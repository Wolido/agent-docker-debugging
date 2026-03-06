# Language-Specific Container Issues

## Rust

### Dependency Version Mismatch

**Error:**
```
error: the lock file needs to be updated but --locked was passed
```

**Cause:** Base image version too old for dependency lock file format

**Solution:**
```dockerfile
# ❌ Old version causing incompatibility
FROM rust:1.70-slim AS builder

# ✅ Update to recent version
FROM rust:1.80-slim AS builder
```

**General Rule:** Keep base image updated, especially for compiled languages

---

### SQLite Static Linking

**Error:**
```
unable to open database file
```

**Cause:** SQLite needs system library or static linking

**Solutions:**

Option 1 - Use bundled/embedded version:
```toml
# Option 1 - Embed database library into binary (static linking)
# Check your database library's documentation for "bundled" or "static" feature
```

Option 2 - Install system library:
```dockerfile
RUN apt-get update && apt-get install -y libsqlite3-dev  # Or your DB library
```

---

### Missing System Libraries

**Error:**
```
openssl-sys: could not find directory of OpenSSL installation
```

**Solution:**
```dockerfile
FROM rust:1.80-slim AS builder
RUN apt-get update && apt-get install -y pkg-config libssl-dev
```

**Common needed packages:** `libssl-dev`, `pkg-config`, `libpq-dev`

---

### WORKDIR Order

**Error:** Files not found at runtime, relative paths break

**Solution:**
```dockerfile
# ❌ Wrong: COPY before WORKDIR
COPY . /app
WORKDIR /app

# ✅ Correct: Set WORKDIR first
WORKDIR /app
COPY . .
```

---

### Configuration Loading Issues

**Issue:** Environment variables not overriding file config

**Common Cause:** Config library doesn't properly merge nested env vars

**Solution - Manual override pattern:**
```rust
let mut config = Config::load()?;

// Manual override for critical env vars
if let Ok(db_url) = std::env::var("DATABASE_URL") {
    config.database.url = db_url;
}
if let Ok(port) = std::env::var("PORT") {
    config.server.port = port.parse()?;
}
```

---

### Path Resolution

**Issue:** Relative paths work locally but fail in container

**Solution:**
```rust
// Use absolute paths or resolve at runtime
let config_dir = std::path::PathBuf::from("/app/config");
// Or
let exe_dir = std::env::current_exe()?.parent().unwrap().to_path_buf();
```

---

## Python

### Virtual Environment in Container

**Issue:** Creating venv inside container is unnecessary

**Solution:**
```dockerfile
# ❌ Don't do this - container is already isolated
RUN python -m venv /venv

# ✅ Install directly
RUN pip install --no-cache-dir -r requirements.txt
```

---

### Slim Image Missing Dependencies

**Error:**
```
ModuleNotFoundError or gcc errors
```

**Solution:**
```dockerfile
FROM python:3.11-slim
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*
```

---

## Node.js

### node_modules in Image

**Issue:** Large image, platform-specific binaries

**Solution:**
```dockerfile
# .dockerignore
node_modules
npm-debug.log
```

```dockerfile
# Multi-stage build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/main.js"]
```

---

## Go

### CGO and Static Linking

**Issue:** Binary won't run in scratch/distroless image

**Solution:**
```dockerfile
# Disable CGO for static binary
FROM golang:1.21-alpine AS builder
ENV CGO_ENABLED=0
RUN go build -o app .

FROM scratch
COPY --from=builder /app/app /app
CMD ["/app"]
```

---

## Network Binding (All Languages)

**Critical:** Must bind to `0.0.0.0` in containers, not `localhost`/`127.0.0.1`

### Rust/Go
```rust
// ❌ Only accessible inside container
.server.bind("127.0.0.1:8080")

// ✅ Accessible from host
.server.bind("0.0.0.0:8080")
```

### Python
```python
# ❌ Default is localhost
app.run(host='127.0.0.1', port=8080)

# ✅ Bind to all interfaces
app.run(host='0.0.0.0', port=8080)
```

### Node.js
```javascript
// ❌ May default to localhost
app.listen(8080);

// ✅ Explicit host
app.listen(8080, '0.0.0.0');
```
