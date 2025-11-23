# Self-Hosted Infrastructure Index

## Overview

This is a comprehensive self-hosted infrastructure setup running multiple containerized services on a local network. The infrastructure uses:

- **Traefik** as a reverse proxy and load balancer
- **Pi-hole** for DNS resolution and ad-blocking
- **Docker Compose** for container orchestration
- **Internal `.lan` domains** for service access
- **HTTPS** with self-signed certificates
- **Keycloak** for SSO/Identity Provider

All services are accessible via internal domains (e.g., `https://grafana.lan`) through Traefik, with DNS resolution handled by Pi-hole.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Local Network (192.168.1.x)              │
│                                                              │
│  ┌──────────────┐         ┌──────────────┐                 │
│  │   Pi-hole    │────────▶│   Traefik    │                 │
│  │  (DNS: 53)   │         │  (80, 443)   │                 │
│  │  192.168.1.7 │         │  192.168.1.7 │                 │
│  └──────────────┘         └──────┬───────┘                 │
│                                   │                          │
│                                   │ Routes to:               │
│         ┌─────────────────────────┼─────────────────────┐   │
│         │                         │                     │   │
│    ┌────▼────┐  ┌──────┐  ┌──────▼────┐  ┌──────────┐ │   │
│    │ Grafana │  │  n8n │  │ Keycloak  │  │ Jenkins  │ │   │
│    └─────────┘  └──────┘  └───────────┘  └──────────┘ │   │
│                                                          │   │
│    ┌──────────┐  ┌──────────┐  ┌──────────┐            │   │
│    │ Portainer│  │ Sterling │  │Networking│            │   │
│    └──────────┘  └──────────┘  └──────────┘            │   │
│                                                          │   │
│    ┌──────────────────────────────────────┐              │   │
│    │      Prometheus Stack               │              │   │
│    │  - Prometheus                       │              │   │
│    │  - Grafana                          │              │   │
│    │  - Node Exporter                    │              │   │
│    │  - cAdvisor                         │              │   │
│    └──────────────────────────────────────┘              │   │
└──────────────────────────────────────────────────────────┘
```

### Network Architecture

- **External Network**: `proxy` (shared across all services)
- **Service-Specific Networks**: 
  - `n8n_network` (n8n + PostgreSQL)
  - `keycloak_network` (Keycloak + PostgreSQL)
  - `monitoring` (Prometheus stack)
- **DNS Server**: Pi-hole at `192.168.1.7`
- **Reverse Proxy**: Traefik (ports 80, 443, 8080)

---

## Services Directory

### 1. **Traefik** (`/traefik`)
**Purpose**: Reverse proxy, load balancer, and SSL termination

- **Image**: `traefik:v2.10`
- **Access**: `https://traefik.lan` (Dashboard)
- **Ports**: 
  - `80` (HTTP → HTTPS redirect)
  - `443` (HTTPS)
  - `8080` (Dashboard)
- **Configuration**:
  - `traefik.yml`: Main configuration
  - `dynamic.yml`: Manual service routes (workaround for Docker provider issues)
  - `certs/`: SSL certificates directory
- **Networks**: `proxy`
- **Features**:
  - Automatic HTTPS redirect
  - Docker provider for service discovery
  - File provider for manual routes
  - Self-signed SSL certificates

### 2. **Pi-hole** (`/pihole`)
**Purpose**: DNS server and ad-blocker

- **Image**: `pihole/pihole:latest`
- **Access**: `https://pihole.lan` or `http://localhost:8081`
- **Ports**: 
  - `53/tcp` (DNS)
  - `53/udp` (DNS)
  - `8081:80` (Web UI fallback)
- **DNS Server IP**: `192.168.1.7`
- **Networks**: `proxy`, `default`
- **Configuration**: 
  - `etc-pihole/pihole.toml`: DNS entries for all `.lan` domains
  - Custom DNS entries point all services to `192.168.1.7`
- **Features**:
  - Ad-blocking
  - DNS resolution for `.lan` domains
  - Web interface for management

### 3. **Prometheus Stack** (`/prometheus`)
**Purpose**: Monitoring and metrics collection

#### Services:
- **Prometheus**: Metrics collection and storage
  - **Image**: `prom/prometheus`
  - **Port**: `9090` (internal)
  - **Config**: `prometheus.yml`
  
- **Grafana**: Visualization and dashboards
  - **Image**: `grafana/grafana`
  - **Access**: `https://grafana.lan`
  - **Port**: `3000` (internal)
  - **Default Credentials**: `admin/admin`
  
- **Node Exporter**: System metrics
  - **Image**: `prom/node-exporter:latest`
  - **Port**: `9100` (internal)
  
- **cAdvisor**: Container metrics
  - **Image**: `gcr.io/cadvisor/cadvisor:latest`
  - **Port**: `8080` (internal)

- **Networks**: `monitoring`, `proxy`
- **Volumes**: `prometheus_data`, `grafana_data`

### 4. **n8n** (`/n8n`)
**Purpose**: Workflow automation and integration platform

- **Image**: `n8nio/n8n:latest`
- **Access**: `https://n8n.lan`
- **Port**: `5678` (internal)
- **Database**: PostgreSQL 14
- **Networks**: `n8n_network`, `proxy`
- **Authentication**: Basic Auth enabled
  - Username: `iemafzalhassan`
  - Password: `v1p3r#9900`
- **Volumes**: 
  - `postgres_data`: Database storage
  - `n8n_data`: n8n data
  - `./local-files`: Local file storage

### 5. **Keycloak** (`/key-clock`)
**Purpose**: Identity and Access Management (SSO)

- **Image**: `quay.io/keycloak/keycloak:latest`
- **Access**: `https://keycloak.lan`
- **Port**: `8080` (internal)
- **Database**: PostgreSQL 14
- **Networks**: `keycloak_network`, `proxy`
- **Default Admin**: `admin/admin` (⚠️ Change immediately!)
- **Features**:
  - OpenID Connect
  - OAuth2
  - User federation
  - SSO for other services
- **Documentation**: See `key-clock/README.md` for detailed setup

### 6. **Jenkins** (`/jenkins`)
**Purpose**: CI/CD automation server

- **Image**: `jenkins/jenkins:lts`
- **Access**: `https://jenkins.lan`
- **Port**: `8080` (internal), `50000` (agent port, exposed)
- **Networks**: `proxy`
- **Volumes**: `jenkins_data`
- **User**: `root` (for Docker socket access)

### 7. **Portainer** (`/portainer`)
**Purpose**: Docker container management UI

- **Image**: `portainer/portainer-ce:latest`
- **Access**: `https://portainer.lan`
- **Port**: `9000` (internal)
- **Networks**: `proxy`
- **Volumes**: `portainer_data`
- **Features**: Docker socket access for container management

### 8. **Sterling PDF** (`/sterling`)
**Purpose**: PDF manipulation and processing tool

- **Image**: `frooodle/s-pdf:latest`
- **Access**: `https://sterling.lan`
- **Port**: `8080` (internal)
- **Networks**: `proxy`
- **Features**: PDF editing, conversion, OCR (optional)

### 9. **Networking Tools** (`/networking-tools`)
**Purpose**: Network diagnostic and utility tools

- **Image**: `lissy93/networking-toolbox:latest`
- **Access**: `https://networking.lan`
- **Port**: `3000` (internal)
- **Networks**: `proxy`
- **Features**: Network testing, diagnostics, utilities

---

## Service Access URLs

All services are accessible via HTTPS on `.lan` domains:

| Service | URL | Port (Internal) | Notes |
|---------|-----|-----------------|-------|
| Traefik Dashboard | `https://traefik.lan` | 8080 | Reverse proxy dashboard |
| Grafana | `https://grafana.lan` | 3000 | Default: admin/admin |
| n8n | `https://n8n.lan` | 5678 | Basic auth enabled |
| Keycloak | `https://keycloak.lan` | 8080 | SSO/Identity Provider |
| Jenkins | `https://jenkins.lan` | 8080 | CI/CD server |
| Portainer | `https://portainer.lan` | 9000 | Docker management |
| Sterling PDF | `https://sterling.lan` | 8080 | PDF tools |
| Networking Tools | `https://networking.lan` | 3000 | Network utilities |
| Pi-hole | `https://pihole.lan` | 80 | DNS/Ad-blocker |

---

## Configuration Files

### Traefik
- `traefik/traefik.yml`: Main Traefik configuration
- `traefik/dynamic.yml`: Manual service routes (IP-based routing)
- `traefik/docker-compose.yml`: Traefik service definition
- `traefik/certs/`: SSL certificates directory

### Pi-hole
- `pihole/etc-pihole/pihole.toml`: DNS configuration and custom hosts
- `pihole/etc-dnsmasq.d/`: Additional dnsmasq configuration
- `pihole/docker-compose.yml`: Pi-hole service definition

### Prometheus
- `prometheus/prometheus.yml`: Prometheus scrape configuration
- `prometheus/docker-compose.yml`: Monitoring stack definition

### Service Configurations
Each service has its own `docker-compose.yml` with:
- Service definitions
- Network configurations
- Volume mappings
- Traefik labels for routing
- Environment variables

---

## Network Configuration

### Docker Networks

1. **`proxy`** (External): Shared network for all services
   ```bash
   docker network create proxy
   ```

2. **`n8n_network`**: Internal network for n8n and PostgreSQL

3. **`keycloak_network`**: Internal network for Keycloak and PostgreSQL

4. **`monitoring`**: Internal network for Prometheus stack

### DNS Configuration

- **DNS Server**: Pi-hole at `192.168.1.7`
- **Domain**: `.lan` (local-only, not resolvable externally)
- **All `.lan` domains resolve to**: `192.168.1.7` (Traefik server)

### Required Setup

1. **Configure device DNS** to use `192.168.1.7`
2. **Disable DNS over HTTPS** in browsers (Brave, Chrome, etc.)
3. **Accept self-signed SSL certificates** when accessing services

---

## Quick Start Guide

### Prerequisites

1. Docker and Docker Compose installed
2. Ports 80, 443, 53 available
3. Network access to `192.168.1.7`

### Initial Setup

1. **Create proxy network**:
   ```bash
   docker network create proxy
   ```

2. **Start Traefik** (must be started first):
   ```bash
   cd traefik
   docker-compose up -d
   ```

3. **Start Pi-hole**:
   ```bash
   cd pihole
   docker-compose up -d
   ```

4. **Start other services**:
   ```bash
   cd prometheus && docker-compose up -d
   cd ../n8n && docker-compose up -d
   cd ../key-clock && docker-compose up -d
   cd ../jenkins && docker-compose up -d
   cd ../portainer && docker-compose up -d
   cd ../sterling && docker-compose up -d
   cd ../networking-tools && docker-compose up -d
   ```

5. **Configure DNS** on your device:
   - Set DNS server to `192.168.1.7`
   - Or configure router to use Pi-hole as DNS

6. **Disable DNS over HTTPS** in your browser

7. **Access services** via `https://<service>.lan`

---

## Service Dependencies

```
Traefik (Core Infrastructure)
├── Pi-hole (DNS Resolution)
└── All Services (Routing)

Prometheus Stack
├── Prometheus (Metrics Collection)
├── Grafana (Visualization)
├── Node Exporter (System Metrics)
└── cAdvisor (Container Metrics)

n8n
├── PostgreSQL (Database)
└── n8n (Application)

Keycloak
├── PostgreSQL (Database)
└── Keycloak (Application)

Other Services (Standalone)
├── Jenkins
├── Portainer
├── Sterling PDF
└── Networking Tools
```

---

## Security Considerations

### Current Security Status

⚠️ **Production Readiness**: This setup is configured for local/development use:

1. **Self-signed SSL certificates** - Browsers will show warnings
2. **Default credentials** - Many services use default passwords
3. **No authentication on Traefik dashboard** - `insecure: true`
4. **Basic auth on n8n** - Consider upgrading to OAuth2 via Keycloak
5. **Keycloak admin password** - Default `admin/admin` (must be changed)

### Recommended Security Improvements

1. **Change all default passwords**
2. **Enable Traefik dashboard authentication**
3. **Use mkcert for trusted local certificates**
4. **Configure Keycloak SSO for all services**
5. **Set up firewall rules**
6. **Regular backups**
7. **Keep images updated**

---

## Troubleshooting

### Common Issues

1. **DNS_PROBE_FINISHED_NXDOMAIN**
   - **Cause**: Device not using Pi-hole DNS
   - **Solution**: Configure DNS to `192.168.1.7` and disable DoH in browser
   - **See**: `docs/DNS_SETUP_GUIDE.md`

2. **Services not accessible**
   - **Check**: Containers are running (`docker ps`)
   - **Check**: Services on `proxy` network (`docker network inspect proxy`)
   - **Check**: Traefik logs (`docker logs traefik`)
   - **See**: `docs/QUICK_FIX.md`

3. **SSL Certificate Warnings**
   - **Expected**: Self-signed certificates show warnings
   - **Solution**: Accept certificate or use mkcert for trusted certs

4. **Traefik not routing**
   - **Check**: `dynamic.yml` has correct IP addresses
   - **Check**: Services have Traefik labels
   - **Check**: Services are on `proxy` network

### Documentation References

- `docs/DNS_SETUP_GUIDE.md` - DNS configuration
- `docs/INTERNAL_DOMAIN_SETUP.md` - Domain setup guide
- `docs/QUICK_FIX.md` - Quick troubleshooting
- `docs/FINAL_FIX_SUMMARY.md` - Previous fixes applied
- `key-clock/README.md` - Keycloak setup and SSO configuration

---

## Backup and Maintenance

### Important Volumes to Backup

- `prometheus_data` - Metrics data
- `grafana_data` - Grafana dashboards and configs
- `jenkins_data` - Jenkins jobs and configs
- `portainer_data` - Portainer settings
- `n8n_data` - n8n workflows
- `postgres_data` (n8n) - n8n database
- `postgres_data` (keycloak) - Keycloak database
- `pihole/etc-pihole/` - Pi-hole configuration

### Backup Commands

```bash
# Backup volumes
docker run --rm -v <volume_name>:/data -v $(pwd):/backup \
  alpine tar czf /backup/<volume_name>.tar.gz /data

# Backup Pi-hole config
tar czf pihole_backup.tar.gz pihole/etc-pihole/
```

---

## Environment Details

- **Host OS**: macOS (darwin 25.1.0)
- **Timezone**: Asia/Kolkata (Pi-hole)
- **Network**: 192.168.1.x
- **DNS Server**: 192.168.1.7 (Pi-hole)
- **Reverse Proxy**: Traefik v2.10
- **Container Runtime**: Docker Compose

---

## Service Status Check

```bash
# Check all containers
docker ps

# Check specific service
docker logs <container_name>

# Check network connectivity
docker network inspect proxy

# Check DNS resolution
nslookup grafana.lan 192.168.1.7

# Check Traefik routes
curl -k https://traefik.lan/api/http/routers
```

---

## Future Enhancements

Potential improvements and additions:

1. **SSL Certificates**: Migrate to mkcert or Let's Encrypt (if external domain)
2. **SSO Integration**: Configure Keycloak SSO for all services
3. **Monitoring Alerts**: Set up Grafana alerting
4. **Backup Automation**: Automated backup scripts
5. **Service Discovery**: Fix Traefik Docker provider for automatic discovery
6. **Authentication**: Add authentication to Traefik dashboard
7. **Log Aggregation**: Add Loki or similar for centralized logging
8. **Additional Services**: Consider adding:
   - GitLab/Gitea (Git hosting)
   - Nextcloud (File storage)
   - Home Assistant (Home automation)
   - Vault (Secrets management)

---

## Directory Structure

```
selfhost/
├── traefik/          # Reverse proxy
├── pihole/           # DNS and ad-blocking
├── prometheus/       # Monitoring stack
├── n8n/              # Workflow automation
├── key-clock/        # SSO/Identity Provider
├── jenkins/          # CI/CD server
├── portainer/        # Docker management
├── sterling/         # PDF tools
├── networking-tools/  # Network utilities
└── docs/             # Documentation
```

---

## Contact and Support

For issues or questions:
1. Check documentation in `docs/` directory
2. Review service-specific README files
3. Check Docker logs for errors
4. Verify network and DNS configuration

---

**Last Updated**: Generated from codebase analysis
**Version**: 1.0

