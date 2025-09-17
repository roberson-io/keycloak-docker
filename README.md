# Keycloak Docker Deployment with TLS

This repository provides an example Keycloak deployment using Docker Compose with nginx reverse proxy, PostgreSQL database, and automatic TLS certificate management via Let's Encrypt.

## Prerequisites

- Docker and Docker Compose installed
- Domain name pointing to your server (A record for your domain)
- Ports 80 and 443 open on your firewall
- Certbot installed (see setup instructions below)

## Certbot Installation

1. **Remove existing Certbot packages** (if any):
   ```bash
   sudo apt-get remove certbot
   ```

2. **Install Certbot via snap**:
   ```bash
   sudo snap install --classic certbot
   ```

3. **Create symlink to enable Certbot command**:
   ```bash
   sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```

## Quick Setup

1. **Clone and Configure**
   ```bash
   git clone git@github.com:roberson-io/keycloak-docker.git
   cd keycloak-docker
   cp example.env .env
   ```

2. **Update Configuration**
   Edit `.env` file and update these critical values:
   ```bash
   # Your domain name
   KEYCLOAK_DOMAIN=your-keycloak-domain.com

   # Secure database password (generate a strong password)
   DB_PASSWORD=your-secure-db-password

   # Secure admin credentials (generate a strong password)
   KEYCLOAK_ADMIN_PASSWORD=your-secure-admin-password

   # Your email for Let's Encrypt
   LETSENCRYPT_EMAIL=your-email@domain.com
   ```

3. **Update nginx Configuration**
   Replace domain placeholder in nginx config:
   ```bash
   sed -i 's/DOMAIN_PLACEHOLDER/your-keycloak-domain.com/g' nginx/keycloak.conf
   ```

4. **Start Services** (initially runs over HTTP):
   ```bash
   # Create webroot directory for ACME challenges
   sudo mkdir -p /var/www/certbot

   # Start the stack (nginx will serve Keycloak over HTTP initially)
   docker compose up -d
   ```

5. **Get SSL Certificate**
   ```bash
   # Use certbot with the running nginx container
   sudo certbot certonly --webroot \
     --webroot-path=/var/www/certbot \
     --email your-email@domain.com \
     --agree-tos --no-eff-email \
     -d your-keycloak-domain.com
   ```

6. **Switch to HTTPS Configuration**
   ```bash
   # Replace domain placeholder in HTTPS config
   sed -i 's/DOMAIN_PLACEHOLDER/your-keycloak-domain.com/g' nginx/keycloak.conf

   # Update docker-compose to use full HTTPS config
   sed -i 's|keycloak-http-only.conf|keycloak.conf|g' docker-compose.yml

   # Restart nginx to enable HTTPS redirect
   docker compose restart nginx
   ```

## Configuration Options

### Security Considerations

#### Admin Access
By default, admin endpoints are accessible but rate-limited. To restrict admin access to specific IP addresses, uncomment and modify these lines in `nginx/keycloak.conf`:

```nginx
location /admin/ {
    # Only allow from specific IP ranges
    allow 192.168.1.0/24;  # Local network
    allow 10.0.0.0/8;      # Private network
    deny all;              # Deny all others

    # ... rest of configuration
}
```

#### Rate Limiting
The nginx configuration includes rate limiting:
- General requests: 10 requests/second
- Login endpoints: 10 requests/minute
- Admin endpoints: 5 requests/minute (burst)

## Operations

### Starting Services
```bash
docker compose up -d
```

### Stopping Services
```bash
docker compose down
```

## Support and Contributing

For issues and feature requests, please use the repository's issue tracker.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.
