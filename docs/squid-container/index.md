# Squid Container

A Squid v7 caching proxy container with SSL/TLS BUMP support for HTTPS interception and caching.

## Features

- Built from Squid v7 source code
- SSL BUMP with on-the-fly certificate generation
- Multi-stage Docker build (builder + runtime on Ubuntu 26.04)
- Non-root execution (proxy user)
- Optimized caching rules for package managers

## Configuration

### Network

| Setting | Value |
|---------|-------|
| Listen port | 8080 (HTTP with SSL bump) |
| Allowed networks | RFC1918 + `fd00::/8` (IPv6 private) |
| Safe ports | 80, 443 |

### Cache Settings

| Setting | Value |
|---------|-------|
| Disk cache size | 40 GB |
| Memory cache | 512 MB |
| Max object size | 1 GB |

### Special Cache Rules

| Content Type | Behavior |
|-------------|----------|
| RPM metadata | 1% freshness, 1440s max |
| DEB packages | 129600s TTL (~36 hours) |
| Packages/Release files | Always revalidate |
| General responses | 10+ minute retention |

### SSL/TLS

- Certificates generated on startup via `security_file_certgen`
- SSL certificate database stored at `/squid-state/ssl_db`
- CA certificate and key expected at `/squid-certs/tls.crt` and `/squid-certs/tls.key`

## Entrypoint

On container startup:

1. Creates/initializes the SSL certificate database
2. Initializes Squid cache directories
3. Tails `access.log` and `store.log` to stdout for container logging
4. Runs Squid in foreground (`-NYCd 1`)

## Building

```bash
docker build -t squid:latest squid-container/
```

## Usage with Kubernetes

Mount your TLS certificate and key as a secret at `/squid-certs/`:

```yaml
volumes:
  - name: squid-certs
    secret:
      secretName: squid-tls
containers:
  - name: squid
    image: squid:latest
    ports:
      - containerPort: 8080
    volumeMounts:
      - name: squid-certs
        mountPath: /squid-certs
        readOnly: true
```

Configure nodes to use the proxy by setting `HTTP_PROXY` and `HTTPS_PROXY` environment variables, and distribute the CA certificate to clients for SSL BUMP trust.
