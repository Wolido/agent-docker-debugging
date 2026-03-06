# Common Container Issues

## Issue: Container Won't Start

**Diagnosis:**
1. `docker logs <container-id>`
2. Check exit code: `docker inspect <container> | grep ExitCode`
3. Verify image exists: `docker images`
4. Check entrypoint: `docker inspect --format='{{.Config.Entrypoint}}' <container>`

**Common Exit Codes:**
- `0`: Normal exit
- `1`: General application error
- `127`: Command not found
- `128+N`: Terminated by signal N
- `137`: Out of memory (SIGKILL)
- `139`: Segmentation fault

**Solutions:**
- Fix application error
- Ensure required files exist
- Check executable permissions
- Verify working directory

---

## Issue: Out of Memory

**Symptoms:** Exit code 137 (SIGKILL)

**Debug:**
```bash
docker stats <container-id>
# Check Memory usage vs limit
```

**Solution:**
```bash
# Increase memory limit
docker run -m 512m <image>

# Check current limit
docker inspect <container> | grep Memory
```

---

## Issue: Port Already in Use

**Error:** `bind: address already in use`

**Debug:**
```bash
docker ps  # Check running containers
netstat -tlnp | grep 8080  # Check port usage
lsof -i :8080  # macOS alternative
```

**Solution:**
```bash
# Use different host port
docker run -p 8081:8080 <image>
```

---

## Issue: Network Issues

**Symptom:** Cannot reach other containers

**Debug:**
```bash
docker network ls
docker inspect <container> | grep IPAddress
docker exec <container> ping <other-container>
```

**Solution:**
```bash
# Create and use shared network
docker network create app-network
docker run --network app-network <image>
```

---

## Issue: "Works Locally, Fails in Docker"

**This is the #1 debugging scenario.**

### Checklist:

```bash
# 1. Environment differences
docker exec <container> env  # Container
env                           # Local

# 2. Path differences
docker exec <container> pwd
# Host path may differ from container path!

# 3. Binary/dependency versions
docker exec <container> which <binary>
docker exec <container> <binary> --version

# 4. Missing system libraries
docker exec <container> ldd <binary> 2>&1 | grep "not found"

# 5. File permissions
docker exec <container> ls -la <path>
ls -la <host-path>
```

### Common Causes:

**Environment Variables:**
- Config loading order differs
- Missing env vars in container
- Env var names typos

**File Paths:**
- Working directory different
- Relative paths break
- Missing files in COPY

**System Dependencies:**
- Missing libraries in slim images
- Different glibc versions
- Architecture mismatch (ARM vs x86)

---

## Issue: Docker Compose Service Fails to Start

**Symptom:** `docker-compose ps` shows Exit or Restarting

**Debug:**
```bash
# Check service logs
docker-compose logs <service-name>

# Check depends_on order
docker-compose config --services

# Test without dependencies
docker-compose up --no-deps <service-name>
```

**Common Fixes:**

```yaml
# Add health checks
services:
  db:
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

---

## Issue: Database Connection Fails

**Symptom:** App can't connect to database in Docker

**Common Mistakes:**

```yaml
# ❌ Wrong: Using localhost
services:
  app:
    environment:
      DATABASE_URL: postgresql://user:pass@localhost:5432/db

# ✅ Correct: Use service name
services:
  app:
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/db
```

**Debug:**
```bash
# Test from app container
docker-compose exec app ping db
docker-compose exec app nc -zv db 5432
```

---

## Issue: SQLite "Unable to Open Database File"

**Symptom:** SQLite works locally, fails in Docker

**Causes:**
1. Path doesn't exist in container
2. Permissions issue
3. Missing directory in volume mount

**Solution:**
```dockerfile
# Ensure directory exists with correct permissions
RUN mkdir -p /data && chmod 777 /data
```

```yaml
# Mount named volume
volumes:
  - sqlite_data:/data

volumes:
  sqlite_data:
```

---

## Issue: Config Not Loading

**Symptom:** App uses defaults instead of config values

**Causes:**
1. Config file not copied to container
2. Wrong path in code
3. Environment variables not merged

**Debug:**
```bash
# Verify file exists
docker exec <container> ls -la /app/config/

# Check env vars
docker exec <container> env | grep CONFIG
```

**Solution:**
```dockerfile
# Copy config in Dockerfile
COPY config/ /app/config/
```

```rust
// Manual override if env vars not auto-loaded
if let Ok(val) = std::env::var("SETTING_NAME") {
    config.setting = val;
}
```

---

## Issue: Health Check Failing

**Symptom:** Container marked as unhealthy

**Debug:**
```bash
# Check health status
docker inspect <container> | grep -A 10 "Health"

# Test manually
docker exec <container> wget --spider http://localhost:8080/health
```

**Common Fixes:**

```dockerfile
# Increase start period for slow-starting apps
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

---

## Issue: Permission Denied

**Symptom:** Can't read/write files, execute binaries

**Debug:**
```bash
# Check current user
docker exec <container> whoami

# Check file ownership
docker exec <container> ls -la <path>

# Check numeric UID/GID
docker exec <container> id
```

**Solution:**

```dockerfile
# Run as non-root user
RUN useradd -m myuser
USER myuser

# Or fix permissions
RUN chmod +x /app/entrypoint.sh
RUN chown -R myuser:myuser /data
```
