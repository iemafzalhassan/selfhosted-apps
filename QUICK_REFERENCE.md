# Quick Reference Guide

## 🚀 Quick Start Commands

```bash
# Create proxy network (first time only)
docker network create proxy

# Start all services
cd traefik && docker-compose up -d
cd ../pihole && docker-compose up -d
cd ../prometheus && docker-compose up -d
cd ../n8n && docker-compose up -d
cd ../key-clock && docker-compose up -d
cd ../jenkins && docker-compose up -d
cd ../portainer && docker-compose up -d
cd ../sterling && docker-compose up -d
cd ../networking-tools && docker-compose up -d

# Check status
docker ps

# View logs
docker logs <container_name>

# Restart service
docker restart <container_name>

# Stop all services
docker-compose down  # in each directory
```

## 🌐 Service URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| Traefik | `https://traefik.lan` | - |
| Grafana | `https://grafana.lan` | admin/admin |
| n8n | `https://n8n.lan` | iemafzalhassan/v1p3r#9900 |
| Keycloak | `https://keycloak.lan` | admin/admin ⚠️ |
| Jenkins | `https://jenkins.lan` | - |
| Portainer | `https://portainer.lan` | - |
| Sterling | `https://sterling.lan` | - |
| Networking | `https://networking.lan` | - |
| Pi-hole | `https://pihole.lan` | - |

## 🔧 DNS Configuration

**Pi-hole DNS Server**: `192.168.1.7`

### macOS DNS Setup:
1. System Settings → Network
2. Select connection → Details → DNS
3. Add: `192.168.1.7`
4. Apply

### Browser DNS over HTTPS:
- **Brave**: Settings → Privacy → Use secure DNS → "With your current service provider"
- **Chrome**: Settings → Security → Use secure DNS → "With your current service provider"

### Test DNS:
```bash
nslookup grafana.lan 192.168.1.7
# Should return: 192.168.1.7
```

## 🐳 Container Names

- `traefik` - Reverse proxy
- `pihole` - DNS server
- `prometheus` - Metrics collection
- `grafana` - Dashboards
- `node-exporter` - System metrics
- `cadvisor` - Container metrics
- `n8n` - Workflow automation
- `keycloak` - SSO provider
- `jenkins` - CI/CD
- `portainer` - Docker UI
- `sterling-pdf` - PDF tools
- `networking-tools` - Network utilities

## 📊 Network Information

- **Proxy Network**: `proxy` (external, shared)
- **DNS IP**: `192.168.1.7`
- **Domain**: `.lan` (local only)

## 🔍 Troubleshooting Commands

```bash
# Check container status
docker ps -a

# View container logs
docker logs <container_name> --tail 50

# Check network connectivity
docker network inspect proxy

# Test DNS resolution
dig @192.168.1.7 grafana.lan

# Flush DNS cache (macOS)
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Check Traefik routes
curl -k https://traefik.lan/api/http/routers

# Restart Pi-hole DNS
docker exec pihole pihole restartdns
```

## 🔐 Security Checklist

- [ ] Change Keycloak admin password
- [ ] Change Grafana admin password
- [ ] Enable Traefik dashboard auth
- [ ] Configure Keycloak SSO for services
- [ ] Set up regular backups
- [ ] Update container images regularly

## 📦 Volume Locations

Volumes are managed by Docker. To backup:

```bash
# List volumes
docker volume ls

# Backup volume
docker run --rm -v <volume_name>:/data -v $(pwd):/backup \
  alpine tar czf /backup/<volume_name>.tar.gz /data
```

## 🛠️ Common Tasks

### Update a service:
```bash
cd <service-directory>
docker-compose pull
docker-compose up -d
```

### View service logs:
```bash
docker logs -f <container_name>
```

### Access container shell:
```bash
docker exec -it <container_name> /bin/sh
```

### Restart all services:
```bash
for dir in */; do cd "$dir" && docker-compose restart && cd ..; done
```

## 📝 Configuration Files

- `traefik/traefik.yml` - Traefik main config
- `traefik/dynamic.yml` - Manual service routes
- `pihole/etc-pihole/pihole.toml` - DNS entries
- `prometheus/prometheus.yml` - Metrics config

## 🔗 Related Documentation

- `INDEX.md` - Complete infrastructure documentation
- `docs/DNS_SETUP_GUIDE.md` - DNS configuration guide
- `docs/INTERNAL_DOMAIN_SETUP.md` - Domain setup
- `docs/QUICK_FIX.md` - Troubleshooting guide
- `key-clock/README.md` - Keycloak SSO setup

---

**Tip**: Bookmark this page for quick access to common commands and URLs!

