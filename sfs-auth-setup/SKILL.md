---
name: sfs-auth-setup
description: Set up JWT-based authentication with role-based access control (RBAC) following SFS patterns for Owner, Admin, Staff, and Analyst roles
---

# SFS Auth Setup Skill

This skill implements secure JWT-based authentication and role-based access control (RBAC) following SmartFlow Systems patterns across the SFS ecosystem.

## When to Use This Skill

Invoke this skill when:
- Adding authentication to a new SFS project
- Implementing role-based access control
- Setting up user registration and login
- Adding OAuth/social login
- Implementing multi-tenant authentication
- Adding invitation/token-based registration

## What This Skill Does

### 1. JWT Authentication
- Token generation and validation
- Refresh token mechanism
- Secure token storage
- Token expiration handling

### 2. Role-Based Access Control (RBAC)
- Owner, Admin, Staff, Analyst roles
- Permission middleware
- Route protection
- Feature gating by role

### 3. User Registration
- Email/password registration
- Email verification
- Token-based invitations
- Social OAuth (Google, GitHub)

### 4. Password Security
- Bcrypt hashing
- Password reset flow
- Password strength validation
- Rate limiting

### 5. Session Management
- Secure cookie handling
- Multi-device support
- Session invalidation
- Activity tracking

## SFS Role Hierarchy

```typescript
export enum UserRole {
  OWNER = 'OWNER',       // Full access, billing
  ADMIN = 'ADMIN',       // User management, settings
  STAFF = 'STAFF',       // Limited access, operations
  ANALYST = 'ANALYST',   // Read-only, analytics
}

export const ROLE_HIERARCHY = {
  OWNER: 4,
  ADMIN: 3,
  STAFF: 2,
  ANALYST: 1,
} as const;

export const ROLE_PERMISSIONS = {
  OWNER: [
    'users:manage',
    'billing:manage',
    'settings:manage',
    'data:read',
    'data:write',
    'data:delete',
    'analytics:view',
  ],
  ADMIN: [
    'users:manage',
    'settings:view',
    'data:read',
    'data:write',
    'analytics:view',
  ],
  STAFF: [
    'data:read',
    'data:write',
    'analytics:view',
  ],
  ANALYST: [
    'data:read',
    'analytics:view',
  ],
} as const;
```

## Installation

```bash
npm install jsonwebtoken bcryptjs
npm install -D @types/jsonwebtoken @types/bcryptjs
npm install express-rate-limit
```

## Environment Configuration

```bash
# .env
JWT_SECRET=your-super-secret-jwt-key-change-this
JWT_REFRESH_SECRET=your-refresh-token-secret
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# OAuth (optional)
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx

# Email
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASSWORD=xxx
FROM_EMAIL=noreply@smartflowsystems.com
```

## JWT Utilities

```typescript
// src/lib/jwt.ts
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN || '15m';
const JWT_REFRESH_EXPIRES_IN = process.env.JWT_REFRESH_EXPIRES_IN || '7d';

export interface JWTPayload {
  userId: string;
  email: string;
  role: string;
  tenantId?: string;
}

export function generateAccessToken(payload: JWTPayload): string {
  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRES_IN,
  });
}

export function generateRefreshToken(payload: JWTPayload): string {
  return jwt.sign(payload, JWT_REFRESH_SECRET, {
    expiresIn: JWT_REFRESH_EXPIRES_IN,
  });
}

export function verifyAccessToken(token: string): JWTPayload {
  return jwt.verify(token, JWT_SECRET) as JWTPayload;
}

export function verifyRefreshToken(token: string): JWTPayload {
  return jwt.verify(token, JWT_REFRESH_SECRET) as JWTPayload;
}

export function generateTokenPair(payload: JWTPayload) {
  return {
    accessToken: generateAccessToken(payload),
    refreshToken: generateRefreshToken(payload),
  };
}
```

## Authentication Middleware

```typescript
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import { verifyAccessToken, JWTPayload } from '../lib/jwt';
import { UserRole, ROLE_HIERARCHY } from '../types/roles';

declare global {
  namespace Express {
    interface Request {
      user?: JWTPayload;
    }
  }
}

export function authenticate(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing or invalid authorization header' });
  }

  const token = authHeader.substring(7);

  try {
    const payload = verifyAccessToken(token);
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
}

export function requireRole(...allowedRoles: UserRole[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const userRole = req.user.role as UserRole;

    if (!allowedRoles.includes(userRole)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
}

export function requireMinRole(minRole: UserRole) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const userRole = req.user.role as UserRole;
    const userRoleLevel = ROLE_HIERARCHY[userRole];
    const minRoleLevel = ROLE_HIERARCHY[minRole];

    if (userRoleLevel < minRoleLevel) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
}

export function requirePermission(...permissions: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const userRole = req.user.role as UserRole;
    const userPermissions = ROLE_PERMISSIONS[userRole] || [];

    const hasPermission = permissions.every(permission =>
      userPermissions.includes(permission)
    );

    if (!hasPermission) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
}
```

## Auth Routes

```typescript
// src/routes/auth.ts
import express from 'express';
import bcrypt from 'bcryptjs';
import { prisma } from '../lib/prisma';
import { generateTokenPair, verifyRefreshToken } from '../lib/jwt';
import { authenticate } from '../middleware/auth';
import { sendVerificationEmail, sendPasswordResetEmail } from '../lib/email';

const router = express.Router();

// Register
router.post('/auth/register', async (req, res) => {
  const { email, password, name } = req.body;

  if (!email || !password) {
    return res.status(400).json({ error: 'Email and password required' });
  }

  // Check if user exists
  const existingUser = await prisma.user.findUnique({ where: { email } });
  if (existingUser) {
    return res.status(400).json({ error: 'User already exists' });
  }

  // Hash password
  const hashedPassword = await bcrypt.hash(password, 10);

  // Create user
  const user = await prisma.user.create({
    data: {
      email,
      password: hashedPassword,
      name,
      role: 'STAFF', // Default role
    },
  });

  // Generate verification token
  const verificationToken = crypto.randomUUID();
  await prisma.verificationToken.create({
    data: {
      userId: user.id,
      token: verificationToken,
      type: 'EMAIL_VERIFICATION',
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours
    },
  });

  // Send verification email
  await sendVerificationEmail(email, verificationToken);

  res.status(201).json({
    message: 'User created. Please check your email to verify.',
  });
});

// Login
router.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await prisma.user.findUnique({ where: { email } });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  const validPassword = await bcrypt.compare(password, user.password);
  if (!validPassword) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  if (!user.emailVerified) {
    return res.status(403).json({ error: 'Please verify your email first' });
  }

  const tokens = generateTokenPair({
    userId: user.id,
    email: user.email,
    role: user.role,
    tenantId: user.tenantId,
  });

  // Save refresh token
  await prisma.refreshToken.create({
    data: {
      userId: user.id,
      token: tokens.refreshToken,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
    },
  });

  res.json({
    accessToken: tokens.accessToken,
    refreshToken: tokens.refreshToken,
    user: {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role,
    },
  });
});

// Refresh token
router.post('/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(400).json({ error: 'Refresh token required' });
  }

  try {
    const payload = verifyRefreshToken(refreshToken);

    // Verify token exists in database
    const storedToken = await prisma.refreshToken.findFirst({
      where: {
        userId: payload.userId,
        token: refreshToken,
        expiresAt: { gt: new Date() },
        revoked: false,
      },
    });

    if (!storedToken) {
      return res.status(401).json({ error: 'Invalid refresh token' });
    }

    // Generate new token pair
    const tokens = generateTokenPair(payload);

    // Revoke old refresh token
    await prisma.refreshToken.update({
      where: { id: storedToken.id },
      data: { revoked: true },
    });

    // Save new refresh token
    await prisma.refreshToken.create({
      data: {
        userId: payload.userId,
        token: tokens.refreshToken,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    });

    res.json(tokens);
  } catch (error) {
    return res.status(401).json({ error: 'Invalid refresh token' });
  }
});

// Logout
router.post('/auth/logout', authenticate, async (req, res) => {
  const { refreshToken } = req.body;

  if (refreshToken) {
    await prisma.refreshToken.updateMany({
      where: { userId: req.user!.userId, token: refreshToken },
      data: { revoked: true },
    });
  }

  res.json({ message: 'Logged out successfully' });
});

// Get current user
router.get('/auth/me', authenticate, async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.user!.userId },
    select: {
      id: true,
      email: true,
      name: true,
      role: true,
      createdAt: true,
    },
  });

  res.json(user);
});

export default router;
```

## Password Reset Flow

```typescript
// Add to auth routes
router.post('/auth/forgot-password', async (req, res) => {
  const { email } = req.body;

  const user = await prisma.user.findUnique({ where: { email } });
  if (!user) {
    // Don't reveal if user exists
    return res.json({ message: 'If the email exists, a reset link will be sent' });
  }

  const resetToken = crypto.randomUUID();
  await prisma.verificationToken.create({
    data: {
      userId: user.id,
      token: resetToken,
      type: 'PASSWORD_RESET',
      expiresAt: new Date(Date.now() + 60 * 60 * 1000), // 1 hour
    },
  });

  await sendPasswordResetEmail(email, resetToken);

  res.json({ message: 'If the email exists, a reset link will be sent' });
});

router.post('/auth/reset-password', async (req, res) => {
  const { token, newPassword } = req.body;

  const verificationToken = await prisma.verificationToken.findFirst({
    where: {
      token,
      type: 'PASSWORD_RESET',
      expiresAt: { gt: new Date() },
    },
  });

  if (!verificationToken) {
    return res.status(400).json({ error: 'Invalid or expired token' });
  }

  const hashedPassword = await bcrypt.hash(newPassword, 10);

  await prisma.user.update({
    where: { id: verificationToken.userId },
    data: { password: hashedPassword },
  });

  await prisma.verificationToken.delete({
    where: { id: verificationToken.id },
  });

  res.json({ message: 'Password reset successful' });
});
```

## Rate Limiting

```typescript
// src/middleware/rateLimiter.ts
import rateLimit from 'express-rate-limit';

export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 requests per window
  message: 'Too many login attempts, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

// Apply to auth routes
import { authLimiter } from '../middleware/rateLimiter';
router.post('/auth/login', authLimiter, async (req, res) => { /* ... */ });
```

## Frontend Integration (React)

```typescript
// src/hooks/useAuth.tsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
  role: string;
}

interface AuthState {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  refreshAccessToken: () => Promise<void>;
}

export const useAuth = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      isAuthenticated: false,

      login: async (email, password) => {
        const response = await fetch('/api/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email, password }),
        });

        if (!response.ok) {
          throw new Error('Login failed');
        }

        const data = await response.json();

        set({
          user: data.user,
          accessToken: data.accessToken,
          refreshToken: data.refreshToken,
          isAuthenticated: true,
        });
      },

      logout: async () => {
        const { refreshToken } = get();

        await fetch('/api/auth/logout', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ refreshToken }),
        });

        set({
          user: null,
          accessToken: null,
          refreshToken: null,
          isAuthenticated: false,
        });
      },

      refreshAccessToken: async () => {
        const { refreshToken } = get();

        const response = await fetch('/api/auth/refresh', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ refreshToken }),
        });

        if (!response.ok) {
          get().logout();
          throw new Error('Token refresh failed');
        }

        const data = await response.json();

        set({
          accessToken: data.accessToken,
          refreshToken: data.refreshToken,
        });
      },
    }),
    {
      name: 'auth-storage',
    }
  )
);
```

## Protected Route Component

```typescript
// src/components/ProtectedRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';
import { UserRole, ROLE_HIERARCHY } from '../types/roles';

interface ProtectedRouteProps {
  children: React.ReactNode;
  requireRole?: UserRole;
  requireMinRole?: UserRole;
}

export function ProtectedRoute({
  children,
  requireRole,
  requireMinRole,
}: ProtectedRouteProps) {
  const { isAuthenticated, user } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (requireRole && user?.role !== requireRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  if (requireMinRole) {
    const userRoleLevel = ROLE_HIERARCHY[user?.role as UserRole];
    const minRoleLevel = ROLE_HIERARCHY[requireMinRole];

    if (userRoleLevel < minRoleLevel) {
      return <Navigate to="/unauthorized" replace />;
    }
  }

  return <>{children}</>;
}
```

## Database Schema

```prisma
model User {
  id              String    @id @default(cuid())
  email           String    @unique
  password        String
  name            String?
  role            String    @default("STAFF")
  emailVerified   Boolean   @default(false)
  tenantId        String?
  refreshTokens   RefreshToken[]
  verificationTokens VerificationToken[]
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt
}

model RefreshToken {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String   @unique
  expiresAt DateTime
  revoked   Boolean  @default(false)
  createdAt DateTime @default(now())
}

model VerificationToken {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String   @unique
  type      String   // EMAIL_VERIFICATION, PASSWORD_RESET, INVITATION
  expiresAt DateTime
  createdAt DateTime @default(now())
}
```

## Completion Checklist

- [ ] JWT utilities implemented
- [ ] Authentication middleware created
- [ ] Auth routes added
- [ ] Password reset flow implemented
- [ ] Rate limiting configured
- [ ] Frontend auth hook created
- [ ] Protected route component built
- [ ] Database schema updated
- [ ] Email verification working
- [ ] Role-based access control tested

## Related SFS Skills
- `sfs-db-prisma` - Database configuration
- `sfs-multi-tenant` - Multi-tenant setup
- `sfs-stripe-integration` - Payment integration

## References
- JWT: https://jwt.io/
- OWASP Auth: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
