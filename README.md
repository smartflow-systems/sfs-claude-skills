# SmartFlow Systems - Claude Skills Library

A comprehensive collection of skills for building SFS ecosystem projects with Claude Code.

## 📚 Skill Catalog

### 🚀 **sfs-repo-setup** - Complete Repository Initialization
Initialize new SFS repositories with standard configurations, CI/CD, theme, and deployment settings.

**Use when:**
- Starting a new SFS project
- Converting existing projects to SFS standards
- Setting up new microservices

**Includes:**
- Git initialization and .gitignore
- GitHub Actions CI/CD workflow
- Health check endpoint
- SFS theme configuration
- Replit deployment setup
- Documentation (README, AGENTS.md)

---

### 🎨 **sfs-theme-enforcer** - Brand Consistency
Apply the signature SmartFlow Systems brown/black/gold theme with Tailwind CSS.

**Use when:**
- Starting new SFS projects
- Updating brand consistency
- Converting client projects to white-label

**Includes:**
- Complete Tailwind configuration
- SFS color palette (brown, black, gold)
- Component styling patterns
- Accessibility validation (WCAG AA)
- Dark mode support

---

### ⚙️ **sfs-ci-config** - GitHub Actions CI/CD
Configure comprehensive GitHub Actions pipeline with testing, health checks, and deployment.

**Use when:**
- Adding CI/CD to new repos
- Updating workflow configurations
- Standardizing deployment

**Includes:**
- Multi-job workflow (lint, test, build, deploy)
- ESLint and Prettier configuration
- Jest testing setup
- Replit deployment automation
- Status badges

---

### 🏥 **sfs-health-check** - Application Monitoring
Implement robust health check endpoints and monitoring infrastructure.

**Use when:**
- Adding health monitoring
- Preparing for production
- Implementing uptime tracking

**Includes:**
- Basic `/health` endpoint
- Detailed health checks (DB, APIs, resources)
- Kubernetes probes (liveness, readiness, startup)
- Prometheus metrics integration
- Health check scripts for CI/CD

---

### 💳 **sfs-stripe-integration** - Payment Processing
Complete Stripe integration with subscriptions, webhooks, and multi-tenant support.

**Use when:**
- Adding payment processing
- Implementing subscriptions
- Setting up multi-tenant billing

**Includes:**
- Stripe SDK configuration
- Subscription management (create, update, cancel)
- Webhook handling (secure verification)
- Stripe Connect (multi-tenant)
- Frontend checkout components (React)
- Freemium tier implementation

---

### 🔐 **sfs-auth-setup** - Authentication & Authorization
JWT-based authentication with role-based access control (RBAC).

**Use when:**
- Adding user authentication
- Implementing RBAC
- Setting up user registration

**Includes:**
- JWT token generation and validation
- SFS role hierarchy (Owner, Admin, Staff, Analyst)
- Password hashing (bcrypt)
- Email verification
- Password reset flow
- Rate limiting
- Frontend auth hooks (React)

---

### 🗄️ **sfs-db-prisma** - Database Configuration
Set up Prisma ORM with SFS standard schema patterns.

**Use when:**
- Adding database support
- Implementing data models
- Setting up migrations

**Includes:**
- Prisma installation and setup
- SFS standard schema (User, Tenant, Subscription)
- Migration scripts
- Seed data
- Prisma Client configuration
- Common query patterns

---

### 🏢 **sfs-multi-tenant** - Multi-Tenant Architecture
Implement multi-tenant architecture with tenant isolation and customization.

**Use when:**
- Building white-label platforms
- Adding multi-tenancy
- Implementing tenant isolation

**Includes:**
- Tenant middleware
- Database isolation patterns
- Subdomain routing
- Tenant-specific settings
- Custom branding per tenant
- Tenant Prisma client extension

---

### 📝 **sfs-readme-gen** - Documentation Generator
Generate comprehensive README files with SFS branding and standards.

**Use when:**
- Creating new project documentation
- Updating existing READMEs
- Standardizing documentation

**Includes:**
- SFS-branded README template
- CI/CD badges
- Installation instructions
- API documentation sections
- Contributing guidelines

---

### 🚢 **sfs-deploy-replit** - Replit Deployment
Configure Replit deployment settings and environment.

**Use when:**
- Setting up Replit deployment
- Configuring environment variables
- Preparing for production deployment

**Includes:**
- `.replit` configuration
- `replit.nix` setup
- Environment variable templates
- Port configuration
- Build and start scripts

---

## 🎯 Quick Start Guide

### 1. Install Skills in Claude Code

Each skill is packaged as a ZIP file. To install:

1. Open Claude Code Settings
2. Navigate to Skills section
3. Click "Upload Skill"
4. Select the desired `.zip` file
5. Skill is now available to invoke

### 2. Using Skills

Invoke skills by name in your conversations with Claude:

```
"Use the sfs-repo-setup skill to initialize this new project"
"Apply the sfs-theme-enforcer skill to ensure brand consistency"
"Set up authentication using the sfs-auth-setup skill"
```

### 3. Skill Combinations

Skills work together. Common workflows:

**New SFS Project Setup:**
1. `sfs-repo-setup` - Initialize repository
2. `sfs-theme-enforcer` - Apply branding
3. `sfs-db-prisma` - Set up database
4. `sfs-auth-setup` - Add authentication
5. `sfs-ci-config` - Configure CI/CD
6. `sfs-health-check` - Add monitoring

**Multi-Tenant SaaS:**
1. `sfs-repo-setup` - Initialize
2. `sfs-db-prisma` - Database setup
3. `sfs-multi-tenant` - Multi-tenancy
4. `sfs-auth-setup` - User authentication
5. `sfs-stripe-integration` - Payment processing
6. `sfs-theme-enforcer` - White-label theming

---

## 📦 Skill Files

All skills are located in `/home/garet/SFS/sfs-claude-skills/`:

```
sfs-claude-skills/
├── sfs-repo-setup.zip (3.2K)
├── sfs-theme-enforcer.zip (4.1K)
├── sfs-ci-config.zip (5.1K)
├── sfs-health-check.zip (5.3K)
├── sfs-stripe-integration.zip (5.5K)
├── sfs-auth-setup.zip (4.8K)
├── sfs-db-prisma.zip (2.2K)
├── sfs-multi-tenant.zip (2.4K)
├── sfs-readme-gen.zip (1.5K)
├── sfs-deploy-replit.zip (870B)
└── README.md (this file)
```

---

## 🎓 Skill Development Philosophy

These skills follow SmartFlow Systems principles:

1. **Consistency** - All projects follow same patterns
2. **Security** - Best practices built-in
3. **Scalability** - Multi-tenant from the start
4. **Automation** - CI/CD and health checks standard
5. **Brand** - Brown/black/gold theme everywhere
6. **Documentation** - Always comprehensive

---

## 🔗 SFS Ecosystem Links

- **Main Site:** [SmartFlowSite](https://github.com/smartflow-systems/SmartFlowSite)
- **All Repositories:** [SFS Organization](https://github.com/smartflow-systems)
- **Knowledge Base:** [sfs-knowledge-base](https://github.com/smartflow-systems/sfs-knowledge-base)

---

## 📊 Technology Standards

All SFS skills assume:

- **Frontend:** React 18+, TypeScript, Tailwind CSS
- **Backend:** Node.js 18+, Express, TypeScript
- **Database:** PostgreSQL (production), SQLite (dev)
- **ORM:** Prisma
- **Auth:** JWT with RBAC
- **Payments:** Stripe
- **Deployment:** Replit, GitHub Actions
- **Testing:** Jest, React Testing Library
- **Linting:** ESLint, Prettier

---

## 🛠️ Troubleshooting

### Skill Not Showing Up
- Ensure ZIP file contains `SKILL.md` at root level
- Check YAML frontmatter is properly formatted
- Restart Claude Code after upload

### Skill Invocation Fails
- Verify skill name matches exactly (case-sensitive)
- Check Claude Code version supports skills
- Review skill logs in settings

### Skills Conflict
- Skills are designed to work together
- If conflicts occur, apply skills in suggested order
- Check for overlapping configurations

---

## 🤝 Contributing

To create new SFS skills:

1. Create folder: `sfs-skill-name/`
2. Add `SKILL.md` with YAML frontmatter:
```yaml
---
name: sfs-skill-name
description: Brief description of what skill does
---
```
3. Write comprehensive skill instructions
4. Test thoroughly across projects
5. Package as ZIP: `zip sfs-skill-name.zip SKILL.md`

---

## 📄 License

MIT License - Part of SmartFlow Systems ecosystem

---

## 🌟 Skill Roadmap

Future skills planned:
- `sfs-testing-suite` - Comprehensive test setup
- `sfs-analytics-integration` - Analytics and tracking
- `sfs-email-service` - Email integration (SendGrid, Mailchimp)
- `sfs-oauth-providers` - Social login (Google, GitHub)
- `sfs-api-docs` - OpenAPI/Swagger documentation
- `sfs-monitoring-alerts` - Error tracking and alerts

---

**Built with ❤️ by SmartFlow Systems**

Last Updated: 2025-11-29
