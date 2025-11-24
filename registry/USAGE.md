# Zot Registry Usage Guide

## Quick Start

### 1. Access the Registry

- **Web UI**: `https://registry.lan`
- **API**: `https://registry.lan/v2/`
- **Credentials**: 
  - Username: `admin` (from `.env` file)
  - Password: `9900` (from `.env` file)

### 2. Configure Docker to Use Your Registry

#### For macOS (Docker Desktop):

1. Open Docker Desktop
2. Go to **Settings** → **Docker Engine**
3. Add to JSON configuration:
```json
{
  "insecure-registries": ["registry.lan"]
}
```
4. Click **Apply & Restart**

#### For Linux:

Edit `/etc/docker/daemon.json`:
```json
{
  "insecure-registries": ["registry.lan"]
}
```

Then restart Docker:
```bash
sudo systemctl restart docker
```

### 3. Login to Registry

```bash
docker login registry.lan
# Username: admin
# Password: 9900
```

### 4. Push Images

```bash
# Tag your image
docker tag myimage:latest registry.lan/myimage:latest

# Push to registry
docker push registry.lan/myimage:latest
```

### 5. Pull Images

```bash
# Pull from registry
docker pull registry.lan/myimage:latest
```

## Common Operations

### List All Repositories

```bash
curl -u admin:9900 https://registry.lan/v2/_catalog
```

### List Tags for a Repository

```bash
curl -u admin:9900 https://registry.lan/v2/myimage/tags/list
```

### Delete an Image

```bash
# Get image digest
DIGEST=$(curl -u admin:9900 -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  -I https://registry.lan/v2/myimage/manifests/latest | grep -i "Docker-Content-Digest" | cut -d' ' -f2 | tr -d '\r')

# Delete image
curl -u admin:9900 -X DELETE https://registry.lan/v2/myimage/manifests/$DIGEST
```

### View Registry Metrics

```bash
curl https://registry.lan/metrics
```

## Features Enabled

- ✅ **Web UI**: Browse images via web interface
- ✅ **Search**: Enhanced search with CVE scanning
- ✅ **Metrics**: Prometheus metrics endpoint
- ✅ **Scrub**: Automatic integrity checks (every 24h)
- ✅ **Garbage Collection**: Automatic cleanup (every 24h)
- ✅ **Deduplication**: Storage deduplication enabled
- ✅ **Authentication**: HTTP Basic Auth (htpasswd)

## Storage

Images are stored in Docker volume `registry_zot_data`. 

### Backup

```bash
docker run --rm -v registry_zot_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/zot-backup.tar.gz -C /data .
```

### Restore

```bash
docker run --rm -v registry_zot_data:/data -v $(pwd):/backup \
  alpine tar xzf /backup/zot-backup.tar.gz -C /data
```

## Troubleshooting

### Cannot Push/Pull Images

1. **Check authentication**:
   ```bash
   docker login registry.lan
   ```

2. **Verify registry is accessible**:
   ```bash
   curl -u admin:9900 https://registry.lan/v2/
   ```

3. **Check Docker daemon configuration**:
   - Ensure `registry.lan` is in `insecure-registries` if using self-signed certificates

### View Logs

```bash
# Zot logs
docker logs zot-registry

# Follow logs
docker logs -f zot-registry
```

### Restart Registry

```bash
cd registry
docker-compose restart
```

## Integration Examples

### Jenkins Pipeline

```groovy
stage('Push Image') {
    steps {
        sh '''
            docker login registry.lan -u admin -p 9900
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
    echo "9900" | docker login registry.lan -u admin --password-stdin

- name: Push Image
  run: |
    docker tag myapp:latest registry.lan/myapp:${{ github.sha }}
    docker push registry.lan/myapp:${{ github.sha }}
```

## References

- [Zot Registry Documentation](https://zotregistry.dev/)
- [Zot Configuration Guide](https://zotregistry.dev/v2.1.0/admin-guide/admin-configuration/)
- [OCI Distribution Specification](https://github.com/opencontainers/distribution-spec)

