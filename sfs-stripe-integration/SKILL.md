---
name: sfs-stripe-integration
description: Integrate Stripe payment processing with subscription management, webhooks, and SFS freemium/multi-tenant patterns
---

# SFS Stripe Integration Skill

This skill implements comprehensive Stripe payment processing following SmartFlow Systems patterns, including subscription management, webhook handling, and multi-tenant payment routing.

## When to Use This Skill

Invoke this skill when:
- Adding payment processing to a new SFS project
- Implementing subscription/freemium models
- Setting up multi-tenant payment routing (Stripe Connect)
- Adding invoice generation and billing
- Implementing usage-based billing
- Migrating from another payment provider

## What This Skill Does

### 1. Stripe SDK Configuration
- Install and configure Stripe SDK
- Set up API keys and webhooks
- Configure environment variables
- Add TypeScript type definitions

### 2. Payment Processing
- One-time payments
- Subscription creation and management
- Usage-based billing
- Invoice generation
- Payment method storage

### 3. Webhook Handling
- Secure webhook verification
- Event processing
- Error handling and retry logic
- Idempotency handling

### 4. Multi-Tenant Support (Stripe Connect)
- Platform/marketplace setup
- Connected account creation
- Payment splitting
- Transfer management

### 5. Subscription Tiers
- Free tier configuration
- Paid tier management
- Feature gating by tier
- Usage limits enforcement

## Installation and Setup

### Install Dependencies
```bash
npm install stripe
npm install -D @types/stripe
```

### Environment Variables
```bash
# .env
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_PUBLISHABLE_KEY=pk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# For Stripe Connect (multi-tenant)
STRIPE_CONNECT_CLIENT_ID=ca_xxx

# Frontend
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_xxx
```

## Stripe Client Configuration

```typescript
// src/lib/stripe.ts
import Stripe from 'stripe';

if (!process.env.STRIPE_SECRET_KEY) {
  throw new Error('STRIPE_SECRET_KEY is required');
}

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2024-11-20.acacia',
  typescript: true,
  appInfo: {
    name: 'SmartFlow Systems',
    version: '1.0.0',
    url: 'https://smartflowsystems.com',
  },
});

// Webhook configuration
export const WEBHOOK_SECRET = process.env.STRIPE_WEBHOOK_SECRET || '';

// Price IDs for different tiers
export const PRICE_IDS = {
  FREE: null,
  STARTER: process.env.STRIPE_PRICE_STARTER,
  PRO: process.env.STRIPE_PRICE_PRO,
  ENTERPRISE: process.env.STRIPE_PRICE_ENTERPRISE,
} as const;
```

## Subscription Management

```typescript
// src/services/subscription.ts
import { stripe, PRICE_IDS } from '../lib/stripe';
import { prisma } from '../lib/prisma';

export type SubscriptionTier = 'FREE' | 'STARTER' | 'PRO' | 'ENTERPRISE';

export interface CreateSubscriptionParams {
  userId: string;
  email: string;
  tier: SubscriptionTier;
  paymentMethodId?: string;
}

export async function createSubscription({
  userId,
  email,
  tier,
  paymentMethodId,
}: CreateSubscriptionParams) {
  // Free tier doesn't require Stripe
  if (tier === 'FREE') {
    return await prisma.user.update({
      where: { id: userId },
      data: {
        subscriptionTier: tier,
        subscriptionStatus: 'active',
      },
    });
  }

  // Get or create Stripe customer
  let customer = await getStripeCustomer(userId);

  if (!customer) {
    customer = await stripe.customers.create({
      email,
      metadata: { userId },
      payment_method: paymentMethodId,
      invoice_settings: {
        default_payment_method: paymentMethodId,
      },
    });

    await prisma.user.update({
      where: { id: userId },
      data: { stripeCustomerId: customer.id },
    });
  }

  // Create subscription
  const subscription = await stripe.subscriptions.create({
    customer: customer.id,
    items: [{ price: PRICE_IDS[tier] }],
    payment_behavior: 'default_incomplete',
    payment_settings: { save_default_payment_method: 'on_subscription' },
    expand: ['latest_invoice.payment_intent'],
    metadata: { userId, tier },
  });

  // Save subscription to database
  await prisma.subscription.create({
    data: {
      userId,
      stripeSubscriptionId: subscription.id,
      stripePriceId: PRICE_IDS[tier]!,
      status: subscription.status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });

  return subscription;
}

export async function cancelSubscription(userId: string) {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: { subscription: true },
  });

  if (!user?.subscription?.stripeSubscriptionId) {
    throw new Error('No active subscription found');
  }

  const subscription = await stripe.subscriptions.cancel(
    user.subscription.stripeSubscriptionId
  );

  await prisma.subscription.update({
    where: { userId },
    data: {
      status: subscription.status,
      canceledAt: new Date(),
    },
  });

  return subscription;
}

export async function updateSubscription(
  userId: string,
  newTier: SubscriptionTier
) {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: { subscription: true },
  });

  if (!user?.subscription?.stripeSubscriptionId) {
    throw new Error('No active subscription found');
  }

  const subscription = await stripe.subscriptions.retrieve(
    user.subscription.stripeSubscriptionId
  );

  const updatedSubscription = await stripe.subscriptions.update(
    subscription.id,
    {
      items: [
        {
          id: subscription.items.data[0].id,
          price: PRICE_IDS[newTier],
        },
      ],
      proration_behavior: 'create_prorations',
    }
  );

  await prisma.subscription.update({
    where: { userId },
    data: {
      stripePriceId: PRICE_IDS[newTier]!,
      status: updatedSubscription.status,
    },
  });

  return updatedSubscription;
}

async function getStripeCustomer(userId: string) {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { stripeCustomerId: true },
  });

  if (!user?.stripeCustomerId) return null;

  try {
    return await stripe.customers.retrieve(user.stripeCustomerId);
  } catch (error) {
    return null;
  }
}
```

## Webhook Handler

```typescript
// src/routes/webhooks.ts
import express from 'express';
import { stripe, WEBHOOK_SECRET } from '../lib/stripe';
import { handleStripeEvent } from '../services/webhookHandler';

const router = express.Router();

router.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  async (req, res) => {
    const sig = req.headers['stripe-signature'];

    if (!sig) {
      return res.status(400).send('Missing stripe-signature header');
    }

    let event: Stripe.Event;

    try {
      event = stripe.webhooks.constructEvent(req.body, sig, WEBHOOK_SECRET);
    } catch (err) {
      console.error('Webhook signature verification failed:', err.message);
      return res.status(400).send(`Webhook Error: ${err.message}`);
    }

    try {
      await handleStripeEvent(event);
      res.json({ received: true });
    } catch (err) {
      console.error('Webhook handler failed:', err);
      res.status(500).send('Webhook handler failed');
    }
  }
);

export default router;
```

## Webhook Event Handler

```typescript
// src/services/webhookHandler.ts
import Stripe from 'stripe';
import { prisma } from '../lib/prisma';

export async function handleStripeEvent(event: Stripe.Event) {
  console.log(`Processing webhook: ${event.type}`);

  switch (event.type) {
    case 'customer.subscription.created':
      await handleSubscriptionCreated(event.data.object);
      break;

    case 'customer.subscription.updated':
      await handleSubscriptionUpdated(event.data.object);
      break;

    case 'customer.subscription.deleted':
      await handleSubscriptionDeleted(event.data.object);
      break;

    case 'invoice.paid':
      await handleInvoicePaid(event.data.object);
      break;

    case 'invoice.payment_failed':
      await handleInvoicePaymentFailed(event.data.object);
      break;

    case 'payment_intent.succeeded':
      await handlePaymentIntentSucceeded(event.data.object);
      break;

    case 'payment_intent.payment_failed':
      await handlePaymentIntentFailed(event.data.object);
      break;

    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
}

async function handleSubscriptionCreated(subscription: Stripe.Subscription) {
  const userId = subscription.metadata.userId;

  await prisma.subscription.upsert({
    where: { stripeSubscriptionId: subscription.id },
    create: {
      userId,
      stripeSubscriptionId: subscription.id,
      stripePriceId: subscription.items.data[0].price.id,
      status: subscription.status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
    update: {
      status: subscription.status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });
}

async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  await prisma.subscription.update({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      status: subscription.status,
      stripePriceId: subscription.items.data[0].price.id,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  await prisma.subscription.update({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      status: 'canceled',
      canceledAt: new Date(),
    },
  });

  // Downgrade user to free tier
  const userId = subscription.metadata.userId;
  await prisma.user.update({
    where: { id: userId },
    data: { subscriptionTier: 'FREE' },
  });
}

async function handleInvoicePaid(invoice: Stripe.Invoice) {
  if (!invoice.subscription) return;

  await prisma.payment.create({
    data: {
      stripeInvoiceId: invoice.id,
      stripeSubscriptionId: invoice.subscription as string,
      amount: invoice.amount_paid,
      currency: invoice.currency,
      status: 'succeeded',
      paidAt: new Date(invoice.status_transitions.paid_at! * 1000),
    },
  });
}

async function handleInvoicePaymentFailed(invoice: Stripe.Invoice) {
  // Send notification to user
  console.error(`Payment failed for invoice ${invoice.id}`);

  // Optionally suspend account after multiple failures
  if (invoice.attempt_count >= 3) {
    const subscription = await prisma.subscription.findFirst({
      where: { stripeSubscriptionId: invoice.subscription as string },
    });

    if (subscription) {
      await prisma.user.update({
        where: { id: subscription.userId },
        data: { subscriptionStatus: 'past_due' },
      });
    }
  }
}

async function handlePaymentIntentSucceeded(paymentIntent: Stripe.PaymentIntent) {
  console.log(`Payment succeeded: ${paymentIntent.id}`);
}

async function handlePaymentIntentFailed(paymentIntent: Stripe.PaymentIntent) {
  console.error(`Payment failed: ${paymentIntent.id}`);
}
```

## Frontend Integration (React)

```typescript
// src/components/CheckoutForm.tsx
import { useState } from 'react';
import { loadStripe } from '@stripe/stripe-js';
import {
  Elements,
  CardElement,
  useStripe,
  useElements,
} from '@stripe/react-stripe-js';

const stripePromise = loadStripe(import.meta.env.VITE_STRIPE_PUBLISHABLE_KEY);

function CheckoutFormInner({ tier }: { tier: string }) {
  const stripe = useStripe();
  const elements = useElements();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) return;

    setLoading(true);
    setError(null);

    const cardElement = elements.getElement(CardElement);
    if (!cardElement) return;

    // Create payment method
    const { error: pmError, paymentMethod } = await stripe.createPaymentMethod({
      type: 'card',
      card: cardElement,
    });

    if (pmError) {
      setError(pmError.message || 'Payment failed');
      setLoading(false);
      return;
    }

    // Create subscription
    try {
      const response = await fetch('/api/subscriptions/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          tier,
          paymentMethodId: paymentMethod.id,
        }),
      });

      const data = await response.json();

      if (data.error) {
        setError(data.error);
        return;
      }

      // Handle 3D Secure if needed
      if (data.clientSecret) {
        const { error: confirmError } = await stripe.confirmCardPayment(
          data.clientSecret
        );

        if (confirmError) {
          setError(confirmError.message || 'Payment confirmation failed');
          return;
        }
      }

      // Success!
      window.location.href = '/dashboard?subscription=success';
    } catch (err) {
      setError('Something went wrong');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="p-4 border border-sfs-brown-200 rounded-lg">
        <CardElement
          options={{
            style: {
              base: {
                fontSize: '16px',
                color: '#2d2416',
                '::placeholder': {
                  color: '#bfa490',
                },
              },
            },
          }}
        />
      </div>

      {error && (
        <div className="p-3 bg-red-50 border border-red-200 rounded text-red-600">
          {error}
        </div>
      )}

      <button
        type="submit"
        disabled={!stripe || loading}
        className="btn-primary w-full"
      >
        {loading ? 'Processing...' : 'Subscribe'}
      </button>
    </form>
  );
}

export function CheckoutForm({ tier }: { tier: string }) {
  return (
    <Elements stripe={stripePromise}>
      <CheckoutFormInner tier={tier} />
    </Elements>
  );
}
```

## Stripe Connect (Multi-Tenant)

```typescript
// src/services/stripeConnect.ts
import { stripe } from '../lib/stripe';
import { prisma } from '../lib/prisma';

export async function createConnectedAccount(userId: string, email: string) {
  const account = await stripe.accounts.create({
    type: 'express',
    email,
    capabilities: {
      card_payments: { requested: true },
      transfers: { requested: true },
    },
    metadata: { userId },
  });

  await prisma.user.update({
    where: { id: userId },
    data: { stripeConnectedAccountId: account.id },
  });

  return account;
}

export async function createAccountLink(accountId: string) {
  const accountLink = await stripe.accountLinks.create({
    account: accountId,
    refresh_url: `${process.env.APP_URL}/settings/payments/refresh`,
    return_url: `${process.env.APP_URL}/settings/payments/success`,
    type: 'account_onboarding',
  });

  return accountLink.url;
}

export async function createTransfer(
  connectedAccountId: string,
  amount: number,
  currency: string = 'usd'
) {
  const transfer = await stripe.transfers.create({
    amount,
    currency,
    destination: connectedAccountId,
  });

  return transfer;
}

// Platform fee (10% in this example)
export async function createPaymentWithPlatformFee(
  amount: number,
  connectedAccountId: string
) {
  const platformFee = Math.floor(amount * 0.1);

  const paymentIntent = await stripe.paymentIntents.create({
    amount,
    currency: 'usd',
    application_fee_amount: platformFee,
    transfer_data: {
      destination: connectedAccountId,
    },
  });

  return paymentIntent;
}
```

## Database Schema (Prisma)

```prisma
// prisma/schema.prisma
model User {
  id                       String        @id @default(cuid())
  email                    String        @unique
  stripeCustomerId         String?       @unique
  stripeConnectedAccountId String?       @unique
  subscriptionTier         String        @default("FREE")
  subscriptionStatus       String        @default("inactive")
  subscription             Subscription?
  payments                 Payment[]
  createdAt                DateTime      @default(now())
  updatedAt                DateTime      @updatedAt
}

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

model Payment {
  id                   String   @id @default(cuid())
  userId               String
  user                 User     @relation(fields: [userId], references: [id])
  stripeInvoiceId      String   @unique
  stripeSubscriptionId String
  amount               Int
  currency             String
  status               String
  paidAt               DateTime
  createdAt            DateTime @default(now())
}
```

## Execution Steps

1. **Install Dependencies**
   ```bash
   npm install stripe @stripe/stripe-js @stripe/react-stripe-js
   npm install -D @types/stripe
   ```

2. **Configure Environment**
   - Add Stripe API keys to `.env`
   - Set up webhook endpoint URL
   - Configure webhook secret

3. **Set Up Database**
   - Add Prisma models
   - Run migrations
   - Seed subscription tiers

4. **Implement Backend**
   - Configure Stripe client
   - Create subscription service
   - Add webhook handler
   - Set up API routes

5. **Build Frontend**
   - Install Stripe Elements
   - Create checkout form
   - Add pricing page
   - Build customer portal

6. **Configure Webhooks**
   - Create webhook endpoint in Stripe Dashboard
   - Test webhook delivery
   - Verify event handling

7. **Test Integration**
   - Test subscription creation
   - Test payment processing
   - Test webhook events
   - Test error handling

## Completion Checklist

After integrating Stripe:
- [ ] Stripe SDK installed and configured
- [ ] Environment variables set
- [ ] Database schema updated
- [ ] Subscription service implemented
- [ ] Webhook handler created
- [ ] Frontend checkout form built
- [ ] Webhooks configured in Stripe
- [ ] Payment flow tested end-to-end
- [ ] Error handling verified
- [ ] Security best practices followed

## Security Best Practices

1. **Never expose secret keys** - Only use publishable key on frontend
2. **Verify webhook signatures** - Always use `constructEvent`
3. **Use HTTPS** - Required for Stripe webhooks
4. **Implement idempotency** - Handle duplicate webhook events
5. **Validate amounts** - Always verify on backend
6. **Store minimal data** - Don't store full card numbers
7. **Use test mode** - Test thoroughly before going live

## Related SFS Skills
- `sfs-auth-setup` - User authentication
- `sfs-db-prisma` - Database configuration
- `sfs-multi-tenant` - Multi-tenant architecture

## References
- Stripe API Docs: https://stripe.com/docs/api
- Stripe Connect: https://stripe.com/docs/connect
- Stripe Elements: https://stripe.com/docs/stripe-js
