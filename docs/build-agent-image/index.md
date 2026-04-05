# Build Agent Image

A custom GitHub Actions Runner Controller (ARC) image for CI/CD pipeline builds.

**Registry**: `quay.io/cyclops-k8s/build-agent-image`

## Image Tags

| Tag | Description |
|-----|-------------|
| `latest` | Most recent build |
| `YYYY-MM-DD` | Date-tagged build |
| `YYYY-MM-DD-{sha}` | Date and commit SHA |

The image is rebuilt weekly (Sundays at 00:00 UTC) with automatic 30-day keepalive commits to ensure freshness.

## Installed Tools

Built on top of the official GitHub Actions Runner Controller image, with these additions:

### System Packages

- bash, python3, curl, wget, jq, yq, pipx

### Kubernetes Tools

| Tool | Installation |
|------|-------------|
| kubectl | Official Kubernetes install script |
| kustomize | Official hack script |
| Helm v3 | Helm install script |
| OpenTofu | Official repository |

### Automation

- Ansible (via pipx with `dnspython` injection)

## Development Container

The repository also includes a devcontainer configuration with:

- Docker-out-of-Docker (DooD) — shares the host Docker daemon
- Requires `/var/run/docker.sock` access

### VS Code Extensions

- Docker
- GitHub Pull Requests
- GitHub Copilot
- GitHub Actions
