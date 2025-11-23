# Keycloak SSO Setup

## Quick Start

1. **Start Keycloak**:
   ```bash
   docker-compose up -d
   ```

2. **Access Admin Console**:
   - URL: `https://keycloak.lan`
   - Username: `admin`
   - Password: `admin` (change immediately!)

3. **Create a Realm**:
   - Recommended: Create a new realm called "selfhost" or "homelab"
   - Or use the default "master" realm

## Default Configuration

- **Database**: PostgreSQL (container: `keycloak-db`)
- **Admin User**: `admin` / `admin` (change after first login!)
- **Hostname**: `keycloak.lan`
- **Port**: 8080 (internal, proxied via Traefik on 443)
- **Proxy Mode**: Edge (optimized for reverse proxy)

## Environment Variables

Key environment variables in `docker-compose.yml`:

- `KC_DB`: Database type (postgres)
- `KC_DB_URL_HOST`: Database hostname
- `KC_DB_USERNAME`: Database username
- `KC_DB_PASSWORD`: Database password
- `KEYCLOAK_ADMIN`: Admin username
- `KEYCLOAK_ADMIN_PASSWORD`: Admin password
- `KC_PROXY`: Proxy mode (edge)
- `KC_PROXY_ADDRESS_FORWARDING`: Enable proxy forwarding

## Common Tasks

### Change Admin Password

**Via UI**:
1. Login to Keycloak Admin Console
2. Go to Users → View all users
3. Click on `admin` user
4. Go to Credentials tab
5. Set new password

**Via CLI**:
```bash
docker exec -it keycloak /opt/keycloak/bin/kcadm.sh config credentials \
  --server http://localhost:8080 --realm master --user admin --password admin

docker exec -it keycloak /opt/keycloak/bin/kcadm.sh set-password \
  -r master -u admin -p <new-password>
```

### Create a Client for SSO

1. Go to Clients → Create Client
2. Fill in:
   - **Client ID**: e.g., `grafana`, `n8n`, `bookstack`
   - **Client Protocol**: `openid-connect`
   - **Root URL**: `https://<service>.lan`
   - **Valid Redirect URIs**: `https://<service>.lan/*`
   - **Web Origins**: `https://<service>.lan`
3. Save
4. Go to Credentials tab and copy the "Secret"

### Create a User

1. Go to Users → Add User
2. Fill in:
   - Username
   - Email
   - First Name / Last Name
3. Save
4. Go to Credentials tab → Set Password
5. Uncheck "Temporary" for permanent password

### Reset Database

⚠️ **WARNING**: This will delete all data!

```bash
docker-compose down -v
docker-compose up -d
```

## Integration Examples

### Grafana OAuth2

**Keycloak Client Settings**:
- Client ID: `grafana`
- Valid Redirect URIs: `https://grafana.lan/*`
- Web Origins: `https://grafana.lan`

**Grafana Environment Variables**:
```yaml
GF_AUTH_GENERIC_OAUTH_ENABLED=true
GF_AUTH_GENERIC_OAUTH_NAME=Keycloak
GF_AUTH_GENERIC_OAUTH_CLIENT_ID=grafana
GF_AUTH_GENERIC_OAUTH_CLIENT_SECRET=<client-secret>
GF_AUTH_GENERIC_OAUTH_SCOPES=openid profile email
GF_AUTH_GENERIC_OAUTH_AUTH_URL=https://keycloak.lan/realms/<realm>/protocol/openid-connect/auth
GF_AUTH_GENERIC_OAUTH_TOKEN_URL=https://keycloak.lan/realms/<realm>/protocol/openid-connect/token
GF_AUTH_GENERIC_OAUTH_API_URL=https://keycloak.lan/realms/<realm>/protocol/openid-connect/userinfo
```

### n8n OAuth2

**Keycloak Client Settings**:
- Client ID: `n8n`
- Valid Redirect URIs: `https://n8n.lan/rest/oauth2-credential/callback`

Configure via n8n UI: Settings → Community Nodes → OAuth2

## Troubleshooting

### Keycloak won't start
```bash
# Check logs
docker logs keycloak

# Check database connection
docker exec keycloak-db pg_isready -U keycloak
```

### Can't access admin console
1. Check DNS: `nslookup keycloak.lan 192.168.1.7`
2. Check Traefik routing: `docker logs traefik`
3. Verify Keycloak is running: `docker ps | grep keycloak`

### SSO redirect loop
- Verify redirect URIs match exactly (including protocol and domain)
- Check client secret is correct
- Ensure realm name is correct in URLs

### Database connection issues
```bash
# Check PostgreSQL logs
docker logs keycloak-db

# Verify network connectivity
docker exec keycloak ping postgres
```

## Backup & Restore

### Backup
```bash
# Backup database
docker exec keycloak-db pg_dump -U keycloak keycloak > keycloak_backup.sql

# Or backup entire volume
docker run --rm -v key-clock_postgres_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/keycloak_db_backup.tar.gz /data
```

### Restore
```bash
# Restore database
docker exec -i keycloak-db psql -U keycloak keycloak < keycloak_backup.sql
```

## Performance Tuning

Default Java options: `-Xms512m -Xmx1024m`

To increase memory:
```yaml
JAVA_OPTS_APPEND: "-Xms1024m -Xmx2048m"
```

## Security Recommendations

1. ✅ Change admin password immediately
2. ✅ Use strong password policies
3. ✅ Enable 2FA for admin users
4. ✅ Use HTTPS only (already configured via Traefik)
5. ✅ Regular backups
6. ✅ Keep Keycloak updated
7. ✅ Use separate realms for different environments
8. ✅ Rotate client secrets regularly

## Useful Links

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/)
- [OpenID Connect Specification](https://openid.net/specs/openid-connect-core-1_0.html)

