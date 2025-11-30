---
name: sfs-multi-tenant
description: Implement multi-tenant architecture with tenant isolation, subdomain routing, and per-tenant customization for SFS white-label platforms
---

# SFS Multi-Tenant Setup Skill

Implement complete multi-tenant architecture following SmartFlow Systems white-label patterns.

## Multi-Tenant Strategies

### 1. Shared Database with Tenant ID (SFS Standard)
```prisma
model Tenant {
  id        String   @id @default(cuid())
  name      String
  domain    String?  @unique
  subdomain String?  @unique
  settings  Json?    // Custom branding, features
  users     User[]
  data      Data[]
  createdAt DateTime @default(now())
}

model User {
  id       String  @id @default(cuid())
  tenantId String
  tenant   Tenant  @relation(fields: [tenantId], references: [id])
  email    String
  // ...

  @@unique([tenantId, email])
}

model Data {
  id       String  @id @default(cuid())
  tenantId String
  tenant   Tenant  @relation(fields: [tenantId], references: [id])
  // ...

  @@index([tenantId])
}
```

## Tenant Middleware

```typescript
// src/middleware/tenant.ts
import { Request, Response, NextFunction } from 'express';
import { prisma } from '../lib/prisma';

declare global {
  namespace Express {
    interface Request {
      tenant?: Tenant;
    }
  }
}

export async function tenantMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Extract tenant from subdomain
  const host = req.headers.host || '';
  const subdomain = host.split('.')[0];

  // Or from custom domain
  const domain = host;

  // Find tenant
  const tenant = await prisma.tenant.findFirst({
    where: {
      OR: [
        { subdomain },
        { domain },
      ],
    },
  });

  if (!tenant) {
    return res.status(404).json({ error: 'Tenant not found' });
  }

  req.tenant = tenant;
  next();
}

// Apply tenant filter to all queries
export function withTenant<T>(tenantId: string, query: T): T & { where: any } {
  return {
    ...query,
    where: {
      ...(query as any).where,
      tenantId,
    },
  };
}
```

## Tenant Isolation

```typescript
// src/lib/tenantPrisma.ts
import { PrismaClient } from '@prisma/client';

export function createTenantClient(tenantId: string) {
  const prisma = new PrismaClient();

  // Add tenant filter to all queries
  return prisma.$extends({
    query: {
      $allModels: {
        async findMany({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
        async findFirst({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
        async findUnique({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
        async create({ args, query }) {
          args.data = { ...args.data, tenantId };
          return query(args);
        },
        async update({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
        async delete({ args, query }) {
          args.where = { ...args.where, tenantId };
          return query(args);
        },
      },
    },
  });
}

// Usage
const tenantPrisma = createTenantClient(req.tenant.id);
const users = await tenantPrisma.user.findMany(); // Automatically filtered
```

## Tenant Customization

```typescript
// src/services/tenant.ts
interface TenantSettings {
  branding: {
    primaryColor: string;
    logo: string;
    favicon: string;
  };
  features: {
    analyticsEnabled: boolean;
    stripeEnabled: boolean;
    customDomain: boolean;
  };
  limits: {
    maxUsers: number;
    maxStorage: number;
  };
}

export async function getTenantSettings(tenantId: string): Promise<TenantSettings> {
  const tenant = await prisma.tenant.findUnique({
    where: { id: tenantId },
  });

  return tenant.settings as TenantSettings;
}

export async function updateTenantSettings(
  tenantId: string,
  settings: Partial<TenantSettings>
) {
  return prisma.tenant.update({
    where: { id: tenantId },
    data: {
      settings: settings as any,
    },
  });
}
```

## Tenant Routes

```typescript
// src/routes/tenant.ts
import express from 'express';
import { tenantMiddleware } from '../middleware/tenant';
import { authenticate, requireRole } from '../middleware/auth';

const router = express.Router();

// Apply tenant middleware to all routes
router.use(tenantMiddleware);

// Get tenant info
router.get('/tenant', authenticate, async (req, res) => {
  res.json(req.tenant);
});

// Update tenant settings (OWNER only)
router.patch(
  '/tenant/settings',
  authenticate,
  requireRole('OWNER'),
  async (req, res) => {
    const updated = await updateTenantSettings(req.tenant.id, req.body);
    res.json(updated);
  }
);

export default router;
```

## Subdomain Routing (Express)

```typescript
// src/index.ts
import express from 'express';
import vhost from 'vhost';

const app = express();

// Main application
const mainApp = express();
mainApp.get('/', (req, res) => res.send('Main Site'));

// Tenant application
const tenantApp = express();
tenantApp.use(tenantMiddleware);
tenantApp.get('/', (req, res) => {
  res.send(`Welcome to ${req.tenant.name}`);
});

// Route based on subdomain
app.use(vhost('smartflowsystems.com', mainApp));
app.use(vhost('*.smartflowsystems.com', tenantApp));

app.listen(5000);
```

## Frontend Tenant Detection

```typescript
// src/hooks/useTenant.ts
import { useEffect, useState } from 'react';

export function useTenant() {
  const [tenant, setTenant] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchTenant() {
      const response = await fetch('/api/tenant');
      const data = await response.json();
      setTenant(data);
      setLoading(false);
    }

    fetchTenant();
  }, []);

  return { tenant, loading };
}
```

## Tenant-Specific Theming

```typescript
// src/components/TenantThemeProvider.tsx
import { useTenant } from '../hooks/useTenant';

export function TenantThemeProvider({ children }) {
  const { tenant } = useTenant();

  useEffect(() => {
    if (tenant?.settings?.branding) {
      document.documentElement.style.setProperty(
        '--primary-color',
        tenant.settings.branding.primaryColor
      );
    }
  }, [tenant]);

  return <>{children}</>;
}
```

## Completion Checklist

- [ ] Tenant model created
- [ ] Tenant middleware implemented
- [ ] Tenant isolation configured
- [ ] Subdomain routing set up
- [ ] Tenant settings management
- [ ] Frontend tenant detection
- [ ] Custom branding support
- [ ] Multi-tenant tested

## Related Skills
- sfs-db-prisma
- sfs-auth-setup
- sfs-stripe-integration
