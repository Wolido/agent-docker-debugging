---
name: container-debugging
description: >
  Debug Docker containers and containerized applications. Diagnose deployment
  issues, container lifecycle problems, resource constraints, and multi-service
  Docker Compose setups. Use when containers won't start, application crashes in
  container, "works on my machine" issues, Docker Compose service failures, or
  pre-deployment verification needed.
---

# Container Debugging

## Table of Contents

- [Overview](#overview)
- [When to Use](#when-to-use)
- [Quick Start](#quick-start)
- [Critical: Pre-Deployment Checklist](#critical-pre-deployment-checklist)
- [Reference Guides](#reference-guides)
- [Best Practices](#best-practices)

## Overview

Container debugging focuses on issues within Docker/Kubernetes environments including resource constraints, networking, and application runtime problems.

**Key Principle:** "Works on my machine" means nothing until it works in Docker. Always verify builds and deployments before declaring completion.

## When to Use

- Container won't start
- Application crashes in container
- Resource limits exceeded
- Network connectivity issues
- Performance problems in containers
- **Docker Compose multi-service failures**
- **"Local works, Docker doesn't" environment differences**
- **Pre-deployment verification**
- **Rust/Go/Python/Node.js specific containerization issues**

## Quick Start

Minimal working example:

```bash
# Check container status
docker ps -a
docker inspect <container-id>
docker stats <container-id>

# View container logs
docker logs <container-id>
docker logs --follow <container-id>  # Real-time
docker logs --tail 100 <container-id>  # Last 100 lines

# Connect to running container
docker exec -it <container-id> /bin/bash
docker exec -it <container-id> sh

# Inspect container details
docker inspect <container-id> | grep -A 5 "State"
docker inspect <container-id> | grep -E "Memory|Cpu"

# Check container processes
docker top <container-id>

# View resource usage
docker stats <container-id>
# Shows: CPU%, Memory usage, Network I/O
```

## Critical: Pre-Deployment Checklist

**ALWAYS run these checks before declaring a Docker setup complete:**

```bash
# 1. Clean build test
docker-compose down -v
docker-compose build --no-cache

# 2. Startup test  
docker-compose up -d
sleep 10  # Wait for services to initialize

# 3. Health verification
docker-compose ps
docker-compose logs --tail 50

# 4. Endpoint test
curl http://localhost:<port>/health || echo "Health check FAILED"
```

See [Pre-Deployment Checklist](references/pre-deployment-checklist.md) for complete workflow.

## Reference Guides

Detailed implementations in the `references/` directory:

```yaml
references:
  - file: docker-debugging-basics.md
    contents: Basic Docker debugging commands
  
  - file: common-container-issues.md
    contents: Common issues and solutions
  
  - file: container-optimization.md
    contents: Resource limits and multi-stage builds
  
  - file: debugging-checklist.md
    contents: Quick checklist for troubleshooting
  
  - file: docker-compose-debugging.md
    contents: Multi-service debugging, network issues
  
  - file: pre-deployment-checklist.md
    contents: Mandatory verification steps before deployment
  
  - file: language-specific-issues.md
    contents: Rust, Go, Python, Node.js containerization gotchas
```

## Best Practices

### ✅ DO

- Follow established patterns and conventions
- Write clean, maintainable code
- Add appropriate documentation
- **Test thoroughly BEFORE deploying - see Pre-Deployment Checklist**
- **Assume "local works" ≠ "Docker works"**
- **Use multi-stage builds for compiled languages**
- **Pin base image versions, avoid `latest` tag**

### ❌ DON'T

- Skip testing or validation
- Ignore error handling
- Hard-code configuration values
- **Assume dependencies work the same in containers**
- **Skip pre-deployment verification**
- **Use outdated base images for compiled languages**
