---
name: execution-containerization
description: >-
  Dockerfile best practices, multi-stage builds, image optimization, and
  container security. Step-by-step procedure for containerizing services.
  Use when creating Dockerfiles, optimizing image size, or reviewing container
  configurations.
---

# Containerization

## Purpose

Step-by-step procedure for building production-quality container images.

## Workflow

### Step 1: Choose Base Image

| Need | Recommended base | Size |
|------|-----------------|------|
| Minimal surface area | distroless or Alpine | 5-50MB |
| Debugging capability | slim variants (debian-slim) | 50-100MB |
| Build tools needed | Full OS (build stage only) | 200MB+ |

### Step 2: Multi-Stage Build

Separate build from runtime:
1. Builder stage: install build deps, compile, run tests
2. Runtime stage: copy only artifacts, minimal dependencies

### Step 3: Layer Optimization

- Order instructions from least to most frequently changing
- Copy dependency files before source code (cache deps layer)
- Combine related RUN commands to reduce layers
- Remove build artifacts and cache in the same layer they're created

### Step 4: Security Hardening

- Run as non-root user
- No secrets in image layers (use build secrets or runtime injection)
- Pin dependency versions (deterministic builds)
- Scan for vulnerabilities (Trivy, Snyk)
- Use `.dockerignore` to exclude sensitive files

### Step 5: Health and Signals

- Define HEALTHCHECK instruction
- Handle SIGTERM for graceful shutdown
- Use exec form for ENTRYPOINT (PID 1 signal handling)

## Checklist

- [ ] Multi-stage build (build ≠ runtime)
- [ ] Non-root user in runtime stage
- [ ] No secrets in image
- [ ] .dockerignore excludes unnecessary files
- [ ] Dependencies pinned to specific versions
- [ ] HEALTHCHECK defined
- [ ] Graceful shutdown on SIGTERM
- [ ] Image scanned for vulnerabilities
- [ ] Layer order optimized for cache hits

## Related Skills

- `infrastructure/knowledge-twelve-factor` — containers enforce 12-factor
- `deployment/execution-cicd-pipeline` — building images in CI
- `security/knowledge-secure-by-design` — container security
