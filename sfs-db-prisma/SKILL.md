---
name: sfs-db-prisma
description: Set up Prisma ORM with database configuration, migrations, and SFS standard models for multi-tenant applications
---

# SFS Database (Prisma) Setup Skill

Configure Prisma ORM with database connections and SFS standard schema patterns.

## Installation

```bash
npm install prisma @prisma/client
npm install -D tsx
npx prisma init
```

## Prisma Schema (SFS Standard)

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// User Model with SFS roles
model User {
  id                       String        @id @default(cuid())
  email                    String        @unique
  password                 String
  name                     String?
  role                     String        @default("STAFF") // OWNER, ADMIN, STAFF, ANALYST
  emailVerified            Boolean       @default(false)

  // Multi-tenant support
  tenantId                 String?
  tenant                   Tenant?       @relation(fields: [tenantId], references: [id])

  // Stripe integration
  stripeCustomerId         String?       @unique
  stripeConnectedAccountId String?       @unique
  subscriptionTier         String        @default("FREE")
  subscriptionStatus       String        @default("inactive")

  // Relations
  subscription             Subscription?
  refreshTokens            RefreshToken[]

  createdAt                DateTime      @default(now())
  updatedAt                DateTime      @updatedAt
}

// Multi-tenant model
model Tenant {
  id        String   @id @default(cuid())
  name      String
  domain    String?  @unique
  settings  Json?
  users     User[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// Subscription model
model Subscription {
  id                   String   @id @default(cuid())
  userId               String   @unique
  user                 User     @relation(fields: [userId], references: [id])
  stripeSubscriptionId String   @unique
  stripePriceId        String
  status               String
  currentPeriodStart   DateTime
  currentPeriodEnd     DateTime
  canceledAt           DateTime?
  createdAt            DateTime @default(now())
  updatedAt            DateTime @updatedAt
}

// Refresh Token model
model RefreshToken {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String   @unique
  expiresAt DateTime
  revoked   Boolean  @default(false)
  createdAt DateTime @default(now())
}
```

## Prisma Client Setup

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

## Environment Configuration

```bash
# .env
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# For SQLite (development)
# DATABASE_URL="file:./dev.db"
```

## Migration Commands

```bash
# Create migration
npx prisma migrate dev --name init

# Apply migrations (production)
npx prisma migrate deploy

# Generate Prisma Client
npx prisma generate

# Open Prisma Studio
npx prisma studio

# Reset database (development only!)
npx prisma migrate reset
```

## Package.json Scripts

```json
{
  "scripts": {
    "db:migrate": "prisma migrate dev",
    "db:generate": "prisma generate",
    "db:studio": "prisma studio",
    "db:seed": "tsx prisma/seed.ts"
  }
}
```

## Seed Data

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

async function main() {
  // Create default tenant
  const tenant = await prisma.tenant.create({
    data: {
      name: 'Default Tenant',
      domain: 'default.smartflowsystems.com',
    },
  });

  // Create owner user
  const hashedPassword = await bcrypt.hash('admin123', 10);

  await prisma.user.create({
    data: {
      email: 'admin@smartflowsystems.com',
      password: hashedPassword,
      name: 'Admin User',
      role: 'OWNER',
      emailVerified: true,
      tenantId: tenant.id,
    },
  });

  console.log('Database seeded successfully');
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

## Common Queries

```typescript
// Find user with relations
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    subscription: true,
    tenant: true,
  },
});

// Create user with subscription
const user = await prisma.user.create({
  data: {
    email: 'user@example.com',
    password: hashedPassword,
    subscription: {
      create: {
        stripeSubscriptionId: 'sub_xxx',
        stripePriceId: 'price_xxx',
        status: 'active',
        currentPeriodStart: new Date(),
        currentPeriodEnd: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
      },
    },
  },
});

// Update with transaction
await prisma.$transaction([
  prisma.user.update({
    where: { id: userId },
    data: { subscriptionStatus: 'active' },
  }),
  prisma.subscription.create({
    data: { /* ... */ },
  }),
]);
```

## Completion Checklist

- [ ] Prisma installed
- [ ] Schema defined
- [ ] Database URL configured
- [ ] Initial migration created
- [ ] Prisma Client generated
- [ ] Seed script created
- [ ] Database seeded
- [ ] Prisma Client exported

## Related Skills
- sfs-auth-setup
- sfs-multi-tenant
- sfs-stripe-integration
