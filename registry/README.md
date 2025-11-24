# Zot OCI Registry

Self-hosted Zot OCI registry for storing and managing Docker/OCI images locally.

## Features

- **OCI-Compliant**: Full OCI distribution specification support
- **Private Image Storage**: Store your Docker/OCI images locally
- **Authentication**: HTTP Basic authentication (htpasswd)
- **HTTPS**: Secured with Traefik SSL termination
- **Image Management**: Push, pull, and delete images
- **Web UI**: Built-in web interface for browsing images
- **Search**: Built-in search functionality with CVE scanning
- **Metrics**: Prometheus metrics endpoint
- **Accessible via**: `https://registry.lan`

## Quick Start

### 1. Setup Authentication

First, edit `.env` file with your desired credentials:

```bash
cd registry
cp .env.example .env
# Edit .env with your username and password
```

Then run the setup script to create authentication:

```bash
./setup-auth.sh
```

**Note**: You need `htpasswd` installed:
- macOS: `brew install httpd`
- Ubuntu/Debian: `sudo apt-get install apache2-utils`
- CentOS/RHEL: `sudo yum install httpd-tools`

### 2. Start the Registry

```bash
cd registry
docker-compose up -d
```

### 3. Verify It's Running

```bash
docker ps | grep zot
curl -u username:password https://registry.lan/v2/
```

## Usage

### Configure Docker to Use Your Registry

#### Option 1: Login via Docker CLI

```bash
# Login to your registry
docker login registry.lan
# Username: (from .env)
# Password: (from .env)
```

#### Option 2: Configure Docker Daemon (for insecure registries)

If you're using self-signed certificates, you may need to configure Docker daemon:

**macOS** (Docker Desktop):
1. Open Docker Desktop
2. Go to Settings → Docker Engine
3. Add to JSON:
```json
{
  "insecure-registries": ["registry.lan"]
}
```
4. Click "Apply & Restart"

**Linux** (`/etc/docker/daemon.json`):
```json
{
  "insecure-registries": ["registry.lan"]
}
```
Then restart Docker: `sudo systemctl restart docker`

### Push Images

```bash
# Tag your image
docker tag myimage:latest registry.lan/myimage:latest

# Push to registry
docker push registry.lan/myimage:latest
```

### Pull Images

```bash
# Pull from registry
docker pull registry.lan/myimage:latest
```

### List Images in Registry

```bash
# List repositories (via API)
curl -u username:password https://registry.lan/v2/_catalog

# List tags for a repository
curl -u username:password https://registry.lan/v2/myimage/tags/list

# Or use the web UI
# Open https://registry.lan in your browser
```

### Delete Images

```bash
# Get image digest
curl -u username:password -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  https://registry.lan/v2/myimage/manifests/latest

# Delete image (replace DIGEST with actual digest)
curl -u username:password -X DELETE \
  https://registry.lan/v2/myimage/manifests/DIGEST
```

## Configuration

### Environment Variables

- `HTTP_ADDRESS`: Address to bind to (default: `0.0.0.0`)
- `HTTP_PORT`: Port to listen on (default: `5000`)
- `AUTH_USER`: Username for authentication (default: `admin`)
- `AUTH_PASS`: Password for authentication (default: `change-this-password`)

### Storage

Images are stored in Docker volume `zot_data`. To backup:

```bash
# Backup
docker run --rm -v registry_zot_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/zot-backup.tar.gz -C /data .

# Restore
docker run --rm -v registry_zot_data:/data -v $(pwd):/backup \
  alpine tar xzf /backup/zot-backup.tar.gz -C /data
```

## Web UI

Zot includes a built-in web UI accessible at `https://registry.lan`. You can:
- Browse repositories
- View image tags
- Search for images
- View CVE scan results

## API Endpoints

Zot implements the [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec).

Common endpoints:
- `GET /v2/` - Check registry availability
- `GET /v2/_catalog` - List all repositories
- `GET /v2/<name>/tags/list` - List tags for a repository
- `GET /v2/<name>/manifests/<reference>` - Get manifest
- `GET /metrics` - Prometheus metrics

## Troubleshooting

### Cannot Push/Pull Images

1. **Check authentication**:
   ```bash
   docker login registry.lan
   ```

2. **Verify registry is accessible**:
   ```bash
   curl -u username:password https://registry.lan/v2/
   ```

3. **Check Docker daemon configuration** (for self-signed certs):
   - Ensure `registry.lan` is in `insecure-registries` if using self-signed certificates

### Authentication Issues

- Regenerate htpasswd file:
  ```bash
  ./setup-auth.sh
  ```
- Check `.env` file has correct credentials

### Storage Issues

- Check volume usage:
  ```bash
  docker volume inspect registry_zot_data
  ```
- Clean up old images manually

## Security Notes

⚠️ **Important Security Considerations**:

1. **Change default credentials** in `.env` file
2. **Use strong passwords** for registry authentication
3. **Keep `.env` file local** (it's in `.gitignore`)
4. **Use HTTPS** (already configured via Traefik)
5. **Consider network isolation** for production use

## Integration with CI/CD

### Jenkins

```groovy
stage('Push Image') {
    steps {
        sh '''
            docker login registry.lan -u ${REGISTRY_USER} -p ${REGISTRY_PASS}
            docker tag myapp:latest registry.lan/myapp:${BUILD_NUMBER}
            docker push registry.lan/myapp:${BUILD_NUMBER}
        '''
    }
}
```

### GitHub Actions

```yaml
- name: Login to Registry
  run: |
    echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login registry.lan -u "${{ secrets.REGISTRY_USERNAME }}" --password-stdin

- name: Push Image
  run: |
    docker tag myapp:latest registry.lan/myapp:${{ github.sha }}
    docker push registry.lan/myapp:${{ github.sha }}
```

## Maintenance

### Monitor Storage

```bash
# Check volume size
docker system df -v | grep zot_data
```

### View Logs

```bash
# Zot logs
docker logs zot-registry

# Audit logs (inside container)
docker exec zot-registry cat /var/lib/zot/audit.log
```

## References

- [Zot Registry Documentation](https://zotregistry.dev/)
- [Zot GitHub Repository](https://github.com/project-zot/zot)
- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec)
