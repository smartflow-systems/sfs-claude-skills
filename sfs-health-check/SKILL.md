---
name: sfs-health-check
description: Add comprehensive health check endpoints and monitoring to ensure application reliability and uptime tracking
---

# SFS Health Check Skill

This skill implements robust health check endpoints and monitoring infrastructure to ensure application reliability, facilitate debugging, and enable automated health monitoring across the SFS ecosystem.

## When to Use This Skill

Invoke this skill when:
- Adding health monitoring to a new service
- Improving observability of existing applications
- Setting up uptime monitoring
- Implementing readiness/liveness probes
- Debugging deployment issues
- Preparing for production deployment

## What This Skill Does

### 1. Basic Health Endpoint
- Create `/health` endpoint returning `{"ok":true}`
- Add timestamp and version information
- Include uptime metrics
- Return appropriate HTTP status codes

### 2. Detailed Health Checks
- Database connectivity verification
- External API availability checks
- File system access validation
- Memory and CPU usage monitoring
- Dependency health verification

### 3. Health Check Script
- Create `scripts/health.sh` for CI/CD
- Add timeout and retry logic
- Implement graceful failure handling
- Support different environments

### 4. Monitoring Integration
- Configure health check intervals
- Set up alerting thresholds
- Add logging for health events
- Support external monitoring tools

### 5. Readiness vs Liveness
- Liveness probe (is app running?)
- Readiness probe (is app ready to serve traffic?)
- Startup probe (has app finished starting?)

## Basic Health Endpoint Implementation

### Express.js (Node.js)
```javascript
// src/routes/health.ts
import express from 'express';

const router = express.Router();
const startTime = Date.now();

router.get('/health', (req, res) => {
  res.status(200).json({
    ok: true,
    timestamp: new Date().toISOString(),
    uptime: Math.floor((Date.now() - startTime) / 1000),
    version: process.env.npm_package_version || '1.0.0',
  });
});

router.get('/health/live', (req, res) => {
  // Liveness check - is the app running?
  res.status(200).json({ ok: true });
});

router.get('/health/ready', async (req, res) => {
  // Readiness check - is the app ready to serve traffic?
  try {
    // Check critical dependencies
    await checkDatabase();
    await checkExternalServices();

    res.status(200).json({ ok: true, ready: true });
  } catch (error) {
    res.status(503).json({
      ok: false,
      ready: false,
      error: error.message,
    });
  }
});

export default router;
```

### FastAPI (Python)
```python
# src/routes/health.py
from fastapi import APIRouter, HTTPException
from datetime import datetime
import time

router = APIRouter()
start_time = time.time()

@router.get("/health")
async def health_check():
    return {
        "ok": True,
        "timestamp": datetime.utcnow().isoformat(),
        "uptime": int(time.time() - start_time),
        "version": "1.0.0"
    }

@router.get("/health/live")
async def liveness_check():
    return {"ok": True}

@router.get("/health/ready")
async def readiness_check():
    try:
        # Check critical dependencies
        await check_database()
        await check_external_services()

        return {"ok": True, "ready": True}
    except Exception as e:
        raise HTTPException(
            status_code=503,
            detail={"ok": False, "ready": False, "error": str(e)}
        )
```

## Detailed Health Check Implementation

```javascript
// src/services/healthCheck.ts
import { prisma } from './database';
import axios from 'axios';

export interface HealthStatus {
  ok: boolean;
  timestamp: string;
  uptime: number;
  version: string;
  checks: {
    database: CheckResult;
    externalAPIs: CheckResult;
    fileSystem: CheckResult;
    memory: CheckResult;
  };
}

interface CheckResult {
  status: 'healthy' | 'degraded' | 'unhealthy';
  responseTime?: number;
  error?: string;
  details?: any;
}

const startTime = Date.now();

export async function performHealthCheck(): Promise<HealthStatus> {
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkExternalAPIs(),
    checkFileSystem(),
    checkMemory(),
  ]);

  const [database, externalAPIs, fileSystem, memory] = checks.map(
    result => result.status === 'fulfilled' ? result.value : {
      status: 'unhealthy',
      error: result.reason?.message || 'Unknown error',
    }
  );

  const allHealthy = [database, externalAPIs, fileSystem, memory].every(
    check => check.status === 'healthy'
  );

  return {
    ok: allHealthy,
    timestamp: new Date().toISOString(),
    uptime: Math.floor((Date.now() - startTime) / 1000),
    version: process.env.npm_package_version || '1.0.0',
    checks: {
      database,
      externalAPIs,
      fileSystem,
      memory,
    },
  };
}

async function checkDatabase(): Promise<CheckResult> {
  const start = Date.now();
  try {
    await prisma.$queryRaw`SELECT 1`;
    return {
      status: 'healthy',
      responseTime: Date.now() - start,
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      responseTime: Date.now() - start,
      error: error.message,
    };
  }
}

async function checkExternalAPIs(): Promise<CheckResult> {
  const start = Date.now();
  const apiChecks = [
    process.env.STRIPE_API_URL,
    process.env.OPENAI_API_URL,
  ].filter(Boolean);

  if (apiChecks.length === 0) {
    return { status: 'healthy', details: 'No external APIs configured' };
  }

  try {
    const results = await Promise.all(
      apiChecks.map(url =>
        axios.get(url, { timeout: 5000 }).catch(() => null)
      )
    );

    const healthyCount = results.filter(Boolean).length;
    const status = healthyCount === apiChecks.length ? 'healthy'
      : healthyCount > 0 ? 'degraded'
      : 'unhealthy';

    return {
      status,
      responseTime: Date.now() - start,
      details: {
        total: apiChecks.length,
        healthy: healthyCount,
      },
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      responseTime: Date.now() - start,
      error: error.message,
    };
  }
}

async function checkFileSystem(): Promise<CheckResult> {
  const start = Date.now();
  try {
    const fs = require('fs').promises;
    const testFile = '/tmp/health-check-test';
    await fs.writeFile(testFile, 'test');
    await fs.readFile(testFile);
    await fs.unlink(testFile);

    return {
      status: 'healthy',
      responseTime: Date.now() - start,
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      responseTime: Date.now() - start,
      error: error.message,
    };
  }
}

async function checkMemory(): Promise<CheckResult> {
  const usage = process.memoryUsage();
  const usedMB = Math.round(usage.heapUsed / 1024 / 1024);
  const totalMB = Math.round(usage.heapTotal / 1024 / 1024);
  const percentage = (usedMB / totalMB) * 100;

  const status = percentage < 80 ? 'healthy'
    : percentage < 90 ? 'degraded'
    : 'unhealthy';

  return {
    status,
    details: {
      used: `${usedMB}MB`,
      total: `${totalMB}MB`,
      percentage: `${percentage.toFixed(2)}%`,
    },
  };
}
```

## Health Check Routes

```javascript
// src/routes/health.ts
import express from 'express';
import { performHealthCheck } from '../services/healthCheck';

const router = express.Router();

// Simple health check
router.get('/health', (req, res) => {
  res.status(200).json({
    ok: true,
    timestamp: new Date().toISOString(),
  });
});

// Detailed health check
router.get('/health/detailed', async (req, res) => {
  const health = await performHealthCheck();
  const statusCode = health.ok ? 200 : 503;
  res.status(statusCode).json(health);
});

// Liveness probe (Kubernetes compatible)
router.get('/health/live', (req, res) => {
  res.status(200).json({ ok: true });
});

// Readiness probe (Kubernetes compatible)
router.get('/health/ready', async (req, res) => {
  const health = await performHealthCheck();
  const statusCode = health.ok ? 200 : 503;
  res.status(statusCode).json({
    ok: health.ok,
    ready: health.ok,
  });
});

// Startup probe
router.get('/health/startup', async (req, res) => {
  // Check if application has finished initialization
  const isReady = await checkApplicationStartup();
  if (isReady) {
    res.status(200).json({ ok: true, started: true });
  } else {
    res.status(503).json({ ok: false, started: false });
  }
});

export default router;
```

## Health Check Script

```bash
#!/bin/bash
# scripts/health.sh
set -euo pipefail

# Configuration
PORT=${PORT:-5000}
HOST=${HOST:-localhost}
ENDPOINT="http://${HOST}:${PORT}/health"
MAX_RETRIES=${MAX_RETRIES:-30}
RETRY_DELAY=${RETRY_DELAY:-2}
TIMEOUT=${TIMEOUT:-10}

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Logging functions
log_info() {
  echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
  echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
  echo -e "${RED}[ERROR]${NC} $1"
}

# Check if server is running
check_health() {
  local url=$1
  local response
  local http_code

  response=$(curl -s -w "\n%{http_code}" --max-time "$TIMEOUT" "$url" 2>/dev/null) || return 1
  http_code=$(echo "$response" | tail -n1)

  if [ "$http_code" -eq 200 ]; then
    echo "$response" | head -n-1
    return 0
  else
    log_error "HTTP $http_code received"
    return 1
  fi
}

# Wait for service to be ready
wait_for_service() {
  log_info "Waiting for service at $ENDPOINT..."

  for i in $(seq 1 "$MAX_RETRIES"); do
    if check_health "$ENDPOINT" > /dev/null 2>&1; then
      log_info "Service is ready!"
      return 0
    fi

    if [ "$i" -eq "$MAX_RETRIES" ]; then
      log_error "Service failed to respond after $MAX_RETRIES attempts"
      return 1
    fi

    log_warn "Attempt $i/$MAX_RETRIES - waiting ${RETRY_DELAY}s..."
    sleep "$RETRY_DELAY"
  done
}

# Perform health check
perform_health_check() {
  log_info "Performing health check on $ENDPOINT..."

  local response
  response=$(check_health "$ENDPOINT")

  if [ $? -eq 0 ]; then
    log_info "Health check passed!"
    echo "$response" | jq '.' 2>/dev/null || echo "$response"
    return 0
  else
    log_error "Health check failed!"
    return 1
  fi
}

# Detailed health check
detailed_health_check() {
  local detailed_endpoint="${ENDPOINT}/detailed"
  log_info "Performing detailed health check on $detailed_endpoint..."

  local response
  response=$(check_health "$detailed_endpoint")

  if [ $? -eq 0 ]; then
    echo "$response" | jq '.' 2>/dev/null || echo "$response"

    # Parse response and check individual components
    local ok
    ok=$(echo "$response" | jq -r '.ok' 2>/dev/null)

    if [ "$ok" = "true" ]; then
      log_info "All health checks passed!"
      return 0
    else
      log_warn "Some health checks failed"
      return 1
    fi
  else
    log_error "Detailed health check failed!"
    return 1
  fi
}

# Main execution
main() {
  local mode=${1:-basic}

  case $mode in
    wait)
      wait_for_service
      ;;
    detailed)
      detailed_health_check
      ;;
    basic|*)
      perform_health_check
      ;;
  esac
}

# Run main function
main "$@"
```

## Health Check Monitoring Configuration

### Prometheus Metrics
```javascript
// src/monitoring/metrics.ts
import { register, Counter, Histogram } from 'prom-client';

export const healthCheckCounter = new Counter({
  name: 'health_check_total',
  help: 'Total number of health checks performed',
  labelNames: ['status', 'endpoint'],
});

export const healthCheckDuration = new Histogram({
  name: 'health_check_duration_seconds',
  help: 'Duration of health check requests',
  labelNames: ['endpoint'],
  buckets: [0.1, 0.5, 1, 2, 5],
});

export function recordHealthCheck(
  endpoint: string,
  status: 'success' | 'failure',
  duration: number
) {
  healthCheckCounter.inc({ status, endpoint });
  healthCheckDuration.observe({ endpoint }, duration);
}

// Metrics endpoint
export async function metricsHandler(req, res) {
  res.set('Content-Type', register.contentType);
  res.send(await register.metrics());
}
```

### Uptime Monitoring (UptimeRobot compatible)
```javascript
// src/routes/uptime.ts
import express from 'express';

const router = express.Router();

router.get('/uptime', (req, res) => {
  // Simple endpoint for uptime monitoring services
  res.status(200).send('OK');
});

export default router;
```

## Docker Health Check

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 5000

# Health check configuration
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD node scripts/docker-health-check.js

CMD ["npm", "start"]
```

```javascript
// scripts/docker-health-check.js
const http = require('http');

const options = {
  host: 'localhost',
  port: process.env.PORT || 5000,
  path: '/health',
  timeout: 5000,
};

const request = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

request.on('error', () => {
  process.exit(1);
});

request.end();
```

## Kubernetes Probes

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sfs-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: sfs-app:latest
        ports:
        - containerPort: 5000

        # Liveness probe - restart container if failing
        livenessProbe:
          httpGet:
            path: /health/live
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        # Readiness probe - remove from load balancer if failing
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2

        # Startup probe - allow slow startup
        startupProbe:
          httpGet:
            path: /health/startup
            port: 5000
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 30
```

## Execution Steps

1. **Create Health Endpoints**
   - Add `/health` basic endpoint
   - Add `/health/detailed` for comprehensive checks
   - Add `/health/live` and `/health/ready` for orchestration

2. **Implement Health Check Service**
   - Create health check functions
   - Add database connectivity check
   - Add external API checks
   - Add system resource checks

3. **Create Health Check Script**
   - Create `scripts/health.sh`
   - Make executable: `chmod +x scripts/health.sh`
   - Test locally

4. **Add Monitoring**
   - Set up metrics collection
   - Configure alerting
   - Add logging

5. **Configure CI/CD**
   - Add health check to deployment pipeline
   - Set up automated monitoring
   - Configure failure alerts

6. **Test Health Checks**
   - Test all endpoints
   - Verify failure scenarios
   - Test timeout handling

## Completion Checklist

After implementing health checks:
- [ ] Basic `/health` endpoint working
- [ ] Detailed health check implemented
- [ ] Liveness and readiness probes added
- [ ] Health check script created and tested
- [ ] Monitoring configured
- [ ] Docker health check added (if applicable)
- [ ] Kubernetes probes configured (if applicable)
- [ ] CI/CD integration complete
- [ ] Documentation updated
- [ ] Alerts configured

## Monitoring Tools Integration

### UptimeRobot
- URL: Your `/health` endpoint
- Interval: 5 minutes
- Alert when: Status code != 200

### Pingdom
- Check type: HTTP
- URL: Your `/health` endpoint
- Check interval: 1 minute

### Datadog
```javascript
// Datadog health check monitoring
const StatsD = require('node-dogstatsd').StatsD;
const dogstatsd = new StatsD();

dogstatsd.gauge('app.health.status', health.ok ? 1 : 0);
```

## Related SFS Skills
- `sfs-ci-config` - CI/CD pipeline integration
- `sfs-repo-setup` - Complete repository setup
- `sfs-deploy-replit` - Deployment configuration

## References
- Health Check RFC: https://tools.ietf.org/id/draft-inadarei-api-health-check-01.html
- Kubernetes Probes: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
