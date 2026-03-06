# Pre-Deployment Checklist

## Critical Principle

**"Works on my machine" means nothing. Verify in Docker before declaring completion.**

This checklist prevents the 1+ hour production debugging scenarios by catching issues early.

## Mandatory Pre-Deployment Verification

### Phase 1: Clean Build Test

```bash
# 1. Stop and remove everything (including volumes)
docker-compose down -v

# 2. Remove all images for clean slate
docker rmi $(docker images -q) 2>/dev/null || true

# 3. Build with no cache (catches dependency issues)
docker-compose build --no-cache

# Expected: Build completes without errors
# ❌ If fails: Fix Dockerfile before proceeding
```

### Phase 2: Startup Test

```bash
# 1. Start all services
docker-compose up -d

# 2. Wait for initialization (adjust time as needed)
sleep 10

# 3. Check all services are running
docker-compose ps

# Expected: All services "Up" or "healthy"
# ❌ If any "Exit" or "Restarting": Check logs immediately
```

### Phase 3: Log Inspection

```bash
# Check for errors in all services
docker-compose logs --tail 50

# Check specific service if issues
docker-compose logs <service-name>

# ❌ Red flags to watch for:
# - "Connection refused"
# - "No such file or directory"
# - "Permission denied"
# - "Module not found"
# - Panic/stack traces
```

### Phase 4: Endpoint Verification

```bash
# Test health endpoints (customize for your app)
curl -f http://localhost:8080/health || echo "API health check FAILED"
curl -f http://localhost:3000/ || echo "Frontend check FAILED"

# ❌ If fails: Port mapping or service binding issue
```

### Phase 5: Integration Test

```bash
# Test actual functionality
curl -X POST http://localhost:8080/api/test \
  -H "Content-Type: application/json" \
  -d '{"test": true}'

# ❌ If fails: Internal service configuration issue
```

## Complete Verification Script

Save this as `verify-deployment.sh`:

```bash
#!/bin/bash
set -e

echo "=== Pre-Deployment Verification ==="

# Clean start
echo "1. Cleaning up..."
docker-compose down -v 2>/dev/null || true

# Build
echo "2. Building images..."
docker-compose build --no-cache

# Start
echo "3. Starting services..."
docker-compose up -d

echo "4. Waiting for services to initialize..."
sleep 15

# Check status
echo "5. Checking service status..."
docker-compose ps

# Check logs for errors
echo "6. Checking for startup errors..."
if docker-compose logs | grep -i "error\|panic\|fatal" | head -5; then
    echo "❌ ERRORS FOUND IN LOGS"
    exit 1
fi

# Health checks (customize ports/paths)
echo "7. Health checks..."
HEALTH_PASSED=true

for endpoint in "http://localhost:8080/health" "http://localhost:3000/"; do
    if curl -sf "$endpoint" > /dev/null 2>&1; then
        echo "  ✓ $endpoint"
    else
        echo "  ❌ $endpoint FAILED"
        HEALTH_PASSED=false
    fi
done

if [ "$HEALTH_PASSED" = false ]; then
    echo "❌ HEALTH CHECKS FAILED"
    docker-compose logs --tail 30
    exit 1
fi

echo ""
echo "✅ ALL CHECKS PASSED - Ready for deployment"
```

## Common Failures & Solutions

### Build Failures

```yaml
build_failures:
  - symptom: "Dependency lock file incompatible"
    cause: Base image version too old
    solution: Update to recent base image
  
  - symptom: "unable to find libssl"
    cause: Missing system deps
    solution: Add apt-get install libssl-dev
  
  - symptom: "Database file not found"
    cause: Path doesn't exist in container
    solution: Use absolute paths or create directory
  
  - symptom: "Module not found"
    cause: Node deps not installed
    solution: Check npm install in Dockerfile
```

### Startup Failures

```yaml
startup_failures:
  - symptom: "Exit 1 immediately"
    cause: Entrypoint error
    solution: Check CMD path, make executable
  
  - symptom: "Exit 137"
    cause: Out of memory
    solution: Increase memory limit
  
  - symptom: "Port binding error"
    cause: Port in use
    solution: Change host port mapping
  
  - symptom: "Connection refused"
    cause: Service not ready
    solution: Add depends_on with condition
```

### Runtime Failures

```yaml
runtime_failures:
  - symptom: "Can't reach database"
    cause: Wrong host
    solution: Use service name, not localhost
  
  - symptom: "Config file not loading"
    cause: Wrong path or missing file
    solution: Check WORKDIR, use absolute paths
  
  - symptom: "Env vars not set"
    cause: Not passed
    solution: Verify compose environment section
  
  - symptom: "Permission denied"
    cause: Wrong user
    solution: Fix Dockerfile permissions
```

## Language-Specific Checks

### Rust
- [ ] Base image version compatible with dependencies
- [ ] Static linking for embedded databases (SQLite)
- [ ] OpenSSL dev libraries installed
- [ ] Binary path correct in CMD

### Python
- [ ] requirements.txt up to date
- [ ] Using slim base images appropriately
- [ ] Virtual env if needed
- [ ] Gunicorn/uwsgi configured

### Node.js
- [ ] node_modules not copied (use .dockerignore)
- [ ] Build stage separate from runtime
- [ ] Dist/build output copied correctly

### Go
- [ ] CGO disabled if not needed: `CGO_ENABLED=0`
- [ ] Binary built for correct architecture
- [ ] Scratch/distroless image for minimal size

## After Successful Verification

```bash
# Clean up test environment
docker-compose down -v

# Now safe to:
# - Commit changes
# - Push to CI/CD
# - Deploy to production
```

## Red Flags - STOP and Fix

- ❌ "It should work" without verification
- ❌ Skipping clean build test
- ❌ Assuming env vars work the same
- ❌ Not testing health endpoints
- ❌ Only testing one service in multi-service setup
