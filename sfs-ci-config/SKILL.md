---
name: sfs-ci-config
description: Configure GitHub Actions CI/CD pipeline with SFS patterns including automated testing, health checks, and Replit deployment
---

# SFS CI/CD Configuration Skill

This skill sets up a complete GitHub Actions CI/CD pipeline following SmartFlow Systems standards, with automated testing, health checks, and seamless Replit deployment.

## When to Use This Skill

Invoke this skill when:
- Adding CI/CD to a new SFS repository
- Updating outdated GitHub Actions workflows
- Migrating from another CI/CD platform to GitHub Actions
- Standardizing deployment across SFS projects
- Setting up automated testing pipelines

## What This Skill Does

### 1. GitHub Actions Workflow Creation
- Create `.github/workflows/sfs-ci-deploy.yml`
- Configure triggers (push, pull request, manual)
- Set up job dependencies and sequencing
- Add environment variable management

### 2. Test Automation
- Configure test runners (Jest, Pytest, etc.)
- Set up code linting and formatting checks
- Add TypeScript type checking
- Configure coverage reporting

### 3. Build Process
- Set up production build pipeline
- Configure bundling and optimization
- Add asset compilation
- Verify build artifacts

### 4. Health Check Integration
- Run health check scripts before deployment
- Validate endpoint availability
- Test response format
- Verify critical functionality

### 5. Deployment Automation
- Configure Replit deployment triggers
- Set up deployment secrets
- Add rollback capabilities
- Configure deployment notifications

### 6. Badge Generation
- Create CI/CD status badges
- Add deployment status indicators
- Configure test coverage badges
- Set up performance metrics badges

## Complete GitHub Actions Workflow

```yaml
name: SFS CI/CD Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  workflow_dispatch:

env:
  NODE_VERSION: '18'
  REPLIT_TOKEN: ${{ secrets.REPLIT_TOKEN }}
  SFS_PAT: ${{ secrets.SFS_PAT }}

jobs:
  # Job 1: Lint and Format Check
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Check formatting
        run: npm run format:check

  # Job 2: Type Check (TypeScript projects)
  typecheck:
    name: Type Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run type check
        run: npm run typecheck

  # Job 3: Unit Tests
  test:
    name: Test
    runs-on: ubuntu-latest
    needs: [lint, typecheck]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          flags: unittests

  # Job 4: Build
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/
          retention-days: 7

  # Job 5: Health Check
  health-check:
    name: Health Check
    runs-on: ubuntu-latest
    needs: [build]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run health check
        run: bash scripts/health.sh

  # Job 6: Deploy to Replit
  deploy:
    name: Deploy to Replit
    runs-on: ubuntu-latest
    needs: [health-check]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to Replit
        run: |
          echo "Deploying to Replit..."
          curl -X POST "https://api.replit.com/v1/repls/${{ secrets.REPLIT_REPL_ID }}/deploy" \
            -H "Authorization: Bearer $REPLIT_TOKEN" \
            -H "Content-Type: application/json" \
            -d '{"branch": "main"}'

      - name: Verify deployment
        run: |
          sleep 10
          HEALTH_URL="${{ secrets.REPLIT_URL }}/health"
          RESPONSE=$(curl -s "$HEALTH_URL")
          if echo "$RESPONSE" | grep -q '"ok":true'; then
            echo "Deployment verified successfully"
          else
            echo "Deployment verification failed"
            exit 1
          fi

  # Job 7: Notify on Failure
  notify-failure:
    name: Notify on Failure
    runs-on: ubuntu-latest
    needs: [deploy]
    if: failure()
    steps:
      - name: Send notification
        run: |
          echo "CI/CD pipeline failed. Check logs for details."
          # Add Slack/Discord webhook here if desired
```

## Required GitHub Secrets

Configure these secrets in repository settings:

### Essential Secrets
```
REPLIT_TOKEN        # Replit API token for deployment
SFS_PAT             # GitHub Personal Access Token
REPLIT_REPL_ID      # Replit project ID
REPLIT_URL          # Deployed Replit URL
```

### Optional Secrets
```
CODECOV_TOKEN       # Code coverage reporting
SLACK_WEBHOOK       # Slack notifications
DISCORD_WEBHOOK     # Discord notifications
SENTRY_DSN          # Error tracking
```

## Package.json Scripts

Add these scripts to enable CI/CD workflow:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,js,jsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,js,jsx,json,css,md}\"",
    "typecheck": "tsc --noEmit",
    "health": "bash scripts/health.sh",
    "ci": "npm run lint && npm run typecheck && npm run test && npm run build"
  }
}
```

## ESLint Configuration

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
    },
  },
  plugins: ['@typescript-eslint', 'react'],
  rules: {
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'react/react-in-jsx-scope': 'off',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
}
```

## Prettier Configuration

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}
```

## Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts?(x)', '**/?(*.)+(spec|test).ts?(x)'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
    '!src/main.tsx',
  ],
  coverageThresholds: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '^@/(.*)$': '<rootDir>/src/$1',
  },
}
```

## Health Check Script

```bash
#!/bin/bash
# scripts/health.sh
set -euo pipefail

PORT=${PORT:-5000}
ENDPOINT="http://localhost:$PORT/health"
MAX_RETRIES=30
RETRY_DELAY=2

echo "Starting application for health check..."

# Start server in background
npm run dev &
SERVER_PID=$!

# Cleanup function
cleanup() {
  echo "Stopping server..."
  kill $SERVER_PID 2>/dev/null || true
}
trap cleanup EXIT

# Wait for server to be ready
echo "Waiting for server to start on port $PORT..."
for i in $(seq 1 $MAX_RETRIES); do
  if curl -s "$ENDPOINT" > /dev/null 2>&1; then
    echo "Server is ready!"
    break
  fi

  if [ $i -eq $MAX_RETRIES ]; then
    echo "Server failed to start within timeout"
    exit 1
  fi

  echo "Attempt $i/$MAX_RETRIES - waiting ${RETRY_DELAY}s..."
  sleep $RETRY_DELAY
done

# Run health check
echo "Running health check on $ENDPOINT..."
RESPONSE=$(curl -s "$ENDPOINT")

if echo "$RESPONSE" | grep -q '"ok":true'; then
  echo "✓ Health check passed: $RESPONSE"
  exit 0
else
  echo "✗ Health check failed: $RESPONSE"
  exit 1
fi
```

## Status Badges for README

Add these badges to your README.md:

```markdown
[![CI/CD](https://github.com/smartflow-systems/REPO_NAME/actions/workflows/sfs-ci-deploy.yml/badge.svg)](https://github.com/smartflow-systems/REPO_NAME/actions/workflows/sfs-ci-deploy.yml)
[![codecov](https://codecov.io/gh/smartflow-systems/REPO_NAME/branch/main/graph/badge.svg)](https://codecov.io/gh/smartflow-systems/REPO_NAME)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
```

## Workflow Optimization Tips

### 1. Caching Dependencies
```yaml
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### 2. Parallel Job Execution
```yaml
jobs:
  lint:
    # runs independently
  test:
    needs: [lint]  # wait for lint
  build:
    needs: [lint]  # runs in parallel with test
```

### 3. Conditional Deployment
```yaml
if: |
  github.ref == 'refs/heads/main' &&
  github.event_name == 'push' &&
  !contains(github.event.head_commit.message, '[skip ci]')
```

## Monitoring and Notifications

### Slack Integration
```yaml
- name: Notify Slack
  if: always()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment ${{ job.status }}'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Discord Integration
```yaml
- name: Notify Discord
  if: failure()
  run: |
    curl -H "Content-Type: application/json" \
      -X POST \
      -d '{"content": "🚨 CI/CD Failed for ${{ github.repository }}"}' \
      ${{ secrets.DISCORD_WEBHOOK }}
```

## Deployment Strategies

### Blue-Green Deployment
```yaml
deploy-blue:
  steps:
    - name: Deploy to blue environment
      run: deploy_to_environment.sh blue

deploy-green:
  needs: [deploy-blue]
  steps:
    - name: Verify blue deployment
      run: verify_deployment.sh blue
    - name: Switch traffic to green
      run: switch_traffic.sh green
```

### Canary Deployment
```yaml
deploy-canary:
  steps:
    - name: Deploy 10% traffic
      run: deploy_canary.sh 10
    - name: Monitor metrics
      run: monitor_for_errors.sh
    - name: Increase to 100%
      run: deploy_canary.sh 100
```

## Execution Steps

1. **Create Workflow Directory**
   ```bash
   mkdir -p .github/workflows
   ```

2. **Generate Workflow File**
   - Create `sfs-ci-deploy.yml`
   - Configure jobs and steps
   - Set up environment variables

3. **Configure Secrets**
   - Add secrets in GitHub repository settings
   - Verify secret names match workflow

4. **Add Scripts**
   - Create `scripts/health.sh`
   - Make executable: `chmod +x scripts/health.sh`
   - Test locally

5. **Update package.json**
   - Add required scripts
   - Install dev dependencies

6. **Configure Linting**
   - Add ESLint config
   - Add Prettier config
   - Test linting locally

7. **Set Up Testing**
   - Configure Jest
   - Write sample tests
   - Verify coverage thresholds

8. **Test Workflow**
   - Commit and push to trigger
   - Monitor GitHub Actions tab
   - Fix any failures

## Troubleshooting Common Issues

### Issue: Workflow not triggering
**Fix:** Check branch names match trigger configuration
```yaml
on:
  push:
    branches: [main]  # Must match exact branch name
```

### Issue: Secrets not found
**Fix:** Verify secrets are set at repository level, not organization level

### Issue: Health check timeout
**Fix:** Increase timeout or reduce startup time
```bash
MAX_RETRIES=60  # Increase from 30
RETRY_DELAY=5   # Increase from 2
```

### Issue: Build cache not working
**Fix:** Ensure cache key includes package-lock.json hash
```yaml
key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

## Completion Checklist

After configuring CI/CD:
- [ ] GitHub Actions workflow created
- [ ] Required secrets configured
- [ ] Scripts added and executable
- [ ] Package.json scripts updated
- [ ] ESLint configuration added
- [ ] Prettier configuration added
- [ ] Jest configuration added
- [ ] Health check script tested
- [ ] First workflow run successful
- [ ] Status badges added to README
- [ ] Deployment verified

## Performance Metrics

Target CI/CD performance:
- Lint: < 1 minute
- Type check: < 2 minutes
- Tests: < 5 minutes
- Build: < 3 minutes
- Health check: < 1 minute
- Deploy: < 2 minutes
- **Total pipeline: < 15 minutes**

## Related SFS Skills
- `sfs-repo-setup` - Complete repository setup
- `sfs-health-check` - Health monitoring only
- `sfs-deploy-replit` - Replit deployment config

## References
- GitHub Actions Docs: https://docs.github.com/en/actions
- Replit API: https://docs.replit.com/hosting/deploying-http-servers
- Jest Docs: https://jestjs.io/docs/getting-started
