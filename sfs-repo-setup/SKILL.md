---
name: sfs-repo-setup
description: Initialize a new SmartFlow Systems repository with standard configurations, CI/CD, theme, and deployment settings
---

# SFS Repository Setup Skill

This skill initializes a new SmartFlow Systems repository with all standard configurations, ensuring consistency across the SFS ecosystem.

## When to Use This Skill

Invoke this skill when:
- Creating a new SFS project or repository
- Converting an existing project to SFS standards
- Setting up a new microservice in the SFS ecosystem
- Initializing a client-specific white-label deployment

## What This Skill Does

### 1. Repository Initialization
- Initialize Git repository if not already done
- Set up .gitignore with Node.js/Python/React patterns
- Create standard SFS directory structure
- Add AGENTS.md file for AI assistant guidance

### 2. CI/CD Configuration
- Create `.github/workflows/sfs-ci-deploy.yml` with SFS patterns
- Configure GitHub Actions for:
  - Automated testing on push/PR
  - Deployment to Replit
  - Health check validation
  - Badge generation
- Set up required secrets: `SFS_PAT`, `REPLIT_TOKEN`

### 3. Health Check System
- Add `/health` endpoint that returns `{"ok":true}`
- Create `scripts/health.sh` for monitoring
- Configure health check in deployment workflow

### 4. Theme Application
- Apply brown/black/gold color palette
- Set up Tailwind CSS configuration with SFS colors
- Add brand assets references
- Configure typography system

### 5. Documentation
- Generate README.md with:
  - Project description
  - Installation instructions
  - CI/CD status badges
  - SFS ecosystem links
  - Development workflow
- Create CONTRIBUTING.md if needed
- Add LICENSE (MIT standard for SFS)

### 6. Replit Configuration
- Create `.replit` configuration file
- Set up `replit.nix` for environment
- Configure port settings (default: 5000)
- Add deployment environment variables

### 7. Package Configuration
- Set up package.json with SFS scripts:
  - `dev` - Development server
  - `build` - Production build
  - `health` - Health check script
  - `sync` - Git synchronization
- Configure appropriate Node.js version (18+)

### 8. Environment Setup
- Create `.env.example` template
- Document required environment variables
- Set up `.env` loading patterns

## Standard Directory Structure

```
project-name/
├── .github/
│   └── workflows/
│       └── sfs-ci-deploy.yml
├── src/
│   ├── components/
│   ├── pages/
│   ├── styles/
│   └── utils/
├── public/
├── scripts/
│   └── health.sh
├── tests/
├── .env.example
├── .gitignore
├── .replit
├── replit.nix
├── package.json
├── tsconfig.json (if TypeScript)
├── tailwind.config.js
├── README.md
├── AGENTS.md
└── LICENSE
```

## Execution Steps

1. **Verify Current State**
   - Check if Git is initialized
   - Identify project type (Node.js, Python, React, etc.)
   - Scan for existing configurations

2. **Create Base Structure**
   - Create missing directories
   - Set up .gitignore
   - Initialize package.json if needed

3. **Add CI/CD Pipeline**
   - Create GitHub Actions workflow
   - Configure deployment triggers
   - Set up health check validation

4. **Implement Health Check**
   - Add endpoint to main server file
   - Create health check script
   - Test endpoint functionality

5. **Apply SFS Theme**
   - Configure Tailwind with brown/black/gold palette
   - Add theme configuration files
   - Set up brand asset references

6. **Generate Documentation**
   - Create comprehensive README
   - Add AGENTS.md with project context
   - Include development instructions

7. **Configure Deployment**
   - Set up Replit configuration
   - Add deployment scripts
   - Configure environment variables

8. **Verify Setup**
   - Run health check
   - Test CI/CD workflow
   - Validate theme application

## SFS Color Palette

```javascript
// Tailwind config
module.exports = {
  theme: {
    extend: {
      colors: {
        sfs: {
          brown: {
            50: '#f8f6f4',
            100: '#e8e2dc',
            200: '#d4c4b8',
            300: '#bfa490',
            400: '#a8846e',
            500: '#8b6f47',
            600: '#6b5435',
            700: '#4a3a24',
            800: '#2d2416',
            900: '#1a140c',
          },
          black: '#0a0908',
          gold: {
            300: '#ffd700',
            400: '#ffcc00',
            500: '#d4af37',
            600: '#b8941e',
          },
        },
      },
    },
  },
}
```

## GitHub Actions Workflow Template

```yaml
name: SFS CI/CD Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REPLIT_TOKEN: ${{ secrets.REPLIT_TOKEN }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
      - run: npm run build

  health-check:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: bash scripts/health.sh

  deploy:
    needs: [test, health-check]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Replit
        run: |
          curl -X POST https://api.replit.com/v1/deploy \
            -H "Authorization: Bearer $REPLIT_TOKEN"
```

## Health Check Script Template

```bash
#!/bin/bash
set -euo pipefail

# Health check script for SFS projects
PORT=${PORT:-5000}
ENDPOINT="http://localhost:$PORT/health"

echo "Running health check on $ENDPOINT..."

# Start server in background
npm run dev &
SERVER_PID=$!

# Wait for server to start
sleep 5

# Check health endpoint
RESPONSE=$(curl -s "$ENDPOINT")

if echo "$RESPONSE" | grep -q '"ok":true'; then
  echo "Health check passed: $RESPONSE"
  kill $SERVER_PID
  exit 0
else
  echo "Health check failed: $RESPONSE"
  kill $SERVER_PID
  exit 1
fi
```

## AGENTS.md Template

```markdown
# Agent Notes for [Project Name]

## Project Overview
[Brief description of project purpose and role in SFS ecosystem]

## Key Files
- `[src/index.ts]` - Main entry point
- `[src/server.ts]` - Express server configuration
- `[.github/workflows/sfs-ci-deploy.yml]` - CI/CD pipeline
- `[scripts/health.sh]` - Health check script

## SFS Standards Applied
- Brown/black/gold theme
- GitHub Actions CI/CD
- Health check endpoint: GET /health
- Replit deployment ready

## Development Workflow
1. Make changes in feature branch
2. Test locally: `npm run dev`
3. Run health check: `npm run health`
4. Push to GitHub (triggers CI/CD)
5. Auto-deploy to Replit on main branch

## Important Patterns
- VERIFY before destructive operations
- Always include UNDO instructions
- Bash scripts use `set -euo pipefail`
- Show file paths in brackets [path/to/file]

## Environment Variables
- `PORT` - Server port (default: 5000)
- `SFS_PAT` - GitHub personal access token
- `REPLIT_TOKEN` - Replit deployment token
- `NODE_ENV` - Environment (development/production)
```

## Completion Checklist

After running this skill, verify:
- [ ] Git repository initialized
- [ ] GitHub Actions workflow created
- [ ] Health check endpoint working
- [ ] SFS theme applied
- [ ] README.md complete with badges
- [ ] AGENTS.md created
- [ ] Replit configuration ready
- [ ] Environment variables documented
- [ ] License added (MIT)
- [ ] .gitignore configured

## Next Steps

After setup, suggest:
1. Push to GitHub: `git push origin main`
2. Configure GitHub secrets (SFS_PAT, REPLIT_TOKEN)
3. Test CI/CD pipeline
4. Deploy to Replit
5. Verify health check endpoint live

## Related SFS Skills
- `sfs-theme-enforcer` - Apply/update SFS theme
- `sfs-ci-config` - Configure CI/CD only
- `sfs-health-check` - Add health monitoring only
- `sfs-deploy-replit` - Replit deployment configuration
