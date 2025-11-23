# Self-Hosted Infrastructure

A comprehensive self-hosted infrastructure setup running multiple containerized services on a local network with Traefik reverse proxy, Pi-hole DNS, and Keycloak SSO.

## 📚 Documentation

- **[INDEX.md](INDEX.md)** - Complete infrastructure documentation and analysis
- **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** - Quick reference guide for common commands and URLs
- **[docs/](docs/)** - Detailed guides and troubleshooting documentation

## 🏗️ Architecture Overview

This infrastructure consists of:

- **Reverse Proxy**: Traefik (routing and SSL termination)
- **DNS**: Pi-hole (DNS resolution and ad-blocking)
- **Monitoring**: Prometheus + Grafana + Node Exporter + cAdvisor
- **Automation**: n8n (workflow automation)
- **CI/CD**: Jenkins
- **Identity**: Keycloak (SSO/Identity Provider)
- **Management**: Portainer (Docker UI)
- **Tools**: Sterling PDF, Networking Tools

All services are accessible via internal `.lan` domains through HTTPS.

## 🚀 Quick Start

1. **Create proxy network**:
   ```bash
   docker network create proxy
   ```

2. **Start services** (see [QUICK_REFERENCE.md](QUICK_REFERENCE.md) for details):
   ```bash
   cd traefik && docker-compose up -d
   cd ../pihole && docker-compose up -d
   # ... start other services
   ```

3. **Configure DNS** on your device to use `192.168.1.7` (Pi-hole)

4. **Access services** via `https://<service>.lan`

## 🌐 Service URLs

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| Traefik | `https://traefik.lan` | - |
| Grafana | `https://grafana.lan` | admin/admin |
| n8n | `https://n8n.lan` | See config |
| Keycloak | `https://keycloak.lan` | admin/admin ⚠️ |
| Jenkins | `https://jenkins.lan` | - |
| Portainer | `https://portainer.lan` | - |
| Sterling PDF | `https://sterling.lan` | - |
| Networking Tools | `https://networking.lan` | - |
| Pi-hole | `https://pihole.lan` | - |

⚠️ **Important**: Change default passwords immediately, especially Keycloak admin password!

## 📁 Directory Structure

```
selfhost/
├── traefik/          # Reverse proxy configuration
├── pihole/           # DNS and ad-blocking
├── prometheus/       # Monitoring stack (Prometheus, Grafana, etc.)
├── n8n/              # Workflow automation
├── key-clock/        # Keycloak SSO/Identity Provider
├── jenkins/          # CI/CD server
├── portainer/        # Docker container management
├── sterling/         # PDF manipulation tools
├── networking-tools/ # Network diagnostic tools
├── docs/             # Documentation and guides
├── INDEX.md          # Complete infrastructure index
└── QUICK_REFERENCE.md # Quick reference guide
```

## 🔧 Requirements

- Docker and Docker Compose
- Ports 80, 443, 53 available
- Network access to `192.168.1.7`
- DNS configured to use Pi-hole (`192.168.1.7`)

## 📖 Getting Started

1. Read the **[INDEX.md](INDEX.md)** for complete documentation
2. Check **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** for common commands
3. Review **[docs/DNS_SETUP_GUIDE.md](docs/DNS_SETUP_GUIDE.md)** for DNS configuration
4. See **[docs/INTERNAL_DOMAIN_SETUP.md](docs/INTERNAL_DOMAIN_SETUP.md)** for domain setup

## 🔐 Security Notes

This setup is configured for **local/development use**:

- Self-signed SSL certificates (browser warnings expected)
- Default credentials on some services
- Traefik dashboard without authentication
- **Change all default passwords before production use**

## 🐛 Troubleshooting

Common issues and solutions:

- **DNS not resolving**: Configure device DNS to `192.168.1.7` and disable DNS over HTTPS in browser
- **Services not accessible**: Check containers are running and on `proxy` network
- **SSL warnings**: Expected with self-signed certificates

See **[docs/](docs/)** directory for detailed troubleshooting guides.

## 📝 Maintenance

- Regular backups of volumes and configurations
- Keep container images updated
- Monitor service health via Grafana
- Review logs regularly

## 🤝 Contributing

This is a personal infrastructure setup. For questions or improvements:

1. Check existing documentation
2. Review service-specific README files
3. Check Docker logs for errors

## 📄 License

Personal use infrastructure setup.

---

**Last Updated**: See INDEX.md for detailed version information

