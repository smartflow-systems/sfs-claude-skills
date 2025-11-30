---
name: sfs-deploy-replit
description: Configure Replit deployment settings with environment variables, port configuration, and health check integration
---

# SFS Replit Deployment Skill

Configure complete Replit deployment setup for SmartFlow Systems projects.

## .replit Configuration

```toml
run = "npm start"
entrypoint = "src/index.ts"
modules = ["nodejs-20"]

[nix]
channel = "stable-23_11"

[deployment]
run = ["sh", "-c", "npm start"]
deploymentTarget = "cloudrun"

[[ports]]
localPort = 5000
externalPort = 80
```

## replit.nix Configuration

```nix
{ pkgs }: {
  deps = [
    pkgs.nodejs-20_x
    pkgs.nodePackages.typescript
    pkgs.nodePackages.npm
  ];
}
```

## Environment Setup

```bash
# .env.example
PORT=5000
NODE_ENV=production
DATABASE_URL=
JWT_SECRET=
STRIPE_SECRET_KEY=
```

## package.json Scripts

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "health": "bash scripts/health.sh"
  }
}
```

## Deployment Checklist

- [ ] .replit file created
- [ ] replit.nix configured
- [ ] Environment variables set in Replit Secrets
- [ ] PORT set to 5000
- [ ] Health check endpoint working
- [ ] Build succeeds
- [ ] Deployment tested

## Replit Secrets

Configure in Replit Secrets tab:
- DATABASE_URL
- JWT_SECRET
- STRIPE_SECRET_KEY
- All other sensitive variables

## Related Skills
- sfs-health-check
- sfs-ci-config
