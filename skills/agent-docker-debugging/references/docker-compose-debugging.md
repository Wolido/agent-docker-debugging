# Docker Compose Debugging

## Multi-Service Debugging Workflow

### 1. Service Startup Order Issues

**Symptom:** Services fail because dependencies not ready

```bash
# Check service status
docker-compose ps

# View logs for specific service
docker-compose logs <service-name>
docker-compose logs -f <service-name>  # Follow mode

# Check all service logs
docker-compose logs --tail 100
```

**Solutions:**
```yaml
# Add health checks and depends_on with condition
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
  
  api:
    depends_on:
      db:
        condition: service_healthy
```

### 2. Network Connectivity Between Services

**Symptom:** Services can't reach each other

```bash
# List networks
docker network ls

# Inspect compose network
docker network inspect <project-name>_default

# Test connectivity from one service to another
docker-compose exec <service1> ping <service2>
docker-compose exec <service1> curl http://<service2>:<port>
```

**Common Issues:**
- Services on different networks
- Wrong service name (use service name, not container name)
- Port not exposed (expose in Dockerfile AND map in compose)

### 3. Volume Mount Issues

**Symptom:** Files not visible, permission errors

```bash
# Check volume mounts
docker inspect <container> | grep -A 20 "Mounts"

# Check actual path inside container
docker-compose exec <service> ls -la <mount-path>

# Verify host path exists
ls -la <host-path>
```

**Fix Permissions:**
```dockerfile
# In Dockerfile, ensure correct ownership
RUN mkdir -p /data && chmod 777 /data
```

### 4. Environment Variable Issues

**Symptom:** App can't read env vars, wrong values

```bash
# Check env vars in container
docker-compose exec <service> env | grep <VAR_NAME>
docker-compose exec <service> printenv

# Verify compose sees the variable
docker-compose config  # Shows interpolated compose file
```

**Common Mistakes:**
```yaml
# ❌ Wrong: Variables not passed
services:
  app:
    environment:
      DATABASE_URL  # Empty if not set in shell

# ✅ Correct: Explicit or from .env
services:
  app:
    environment:
      DATABASE_URL: ${DATABASE_URL}  # From shell/.env
    env_file:
      - .env  # Load from file
```

### 5. "Local Works, Docker Doesn't" Investigation

**Checklist:**

```bash
# 1. Compare environments
docker-compose exec <service> env  # Container env
env                                # Local env

# 2. Check file paths
docker-compose exec <service> pwd
# Host path may differ from container path!

# 3. Verify dependencies
docker-compose exec <service> which <binary>
docker-compose exec <service> <binary> --version

# 4. Check for missing system libraries
docker-compose exec <service> ldd <binary> 2>&1 | grep "not found"
```

### 6. Build Context Issues

**Symptom:** Files not found during build, wrong paths

```bash
# Check build context
docker-compose build --no-cache --progress=plain 2>&1

# Verify COPY paths work
docker-compose build --no-cache
```

**Fix PATH Issues:**
```dockerfile
# ❌ Wrong: Relative paths may fail
COPY ./config /app/config

# ✅ Correct: Use WORKDIR first
WORKDIR /app
COPY config/ ./config/
```

### 7. Quick Compose Diagnostics

```bash
# Full restart with clean state
docker-compose down -v
docker-compose up --build -d

# Check service dependencies
docker-compose config --services

# Validate compose file
docker-compose config  # Shows parsed config with env vars

# Single service rebuild
docker-compose up -d --no-deps --build <service-name>

# Scale a service
docker-compose up -d --scale <service>=3
```

## Common Compose Error Messages

```yaml
errors:
  - error: "bind: address already in use"
    cause: Port conflict
    solution: Change host port mapping
  
  - error: "Container is unhealthy"
    cause: Health check failing
    solution: Check logs, adjust health check
  
  - error: "No such container"
    cause: Service name typo
    solution: Use docker-compose config --services
  
  - error: "Network not found"
    cause: Network removed
    solution: docker-compose down then up
  
  - error: "Volume is in use"
    cause: Container using volume
    solution: Stop all containers first
```

## Exit Codes Reference

```yaml
exit_codes:
  - code: 0
    meaning: Success
    cause: Normal exit
  
  - code: 1
    meaning: General error
    cause: App exception
  
  - code: 126
    meaning: Command not executable
    cause: Permission denied
  
  - code: 127
    meaning: Command not found
    cause: Missing binary
  
  - code: 137
    meaning: SIGKILL (128+9)
    cause: Out of memory
  
  - code: 139
    meaning: SIGSEGV (128+11)
    cause: Segmentation fault
  
  - code: 143
    meaning: SIGTERM (128+15)
    cause: Graceful shutdown
```
