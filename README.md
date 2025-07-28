# DNS Proxy with SSL

A secure DNS proxy setup using Pi-hole, Unbound, and Traefik with automatic SSL certificate management.

## 🏗️ Architecture

- **Unbound**: Recursive DNS resolver for enhanced privacy
- **Pi-hole**: Network-wide ad blocking and DNS filtering
- **Traefik**: Reverse proxy with automatic Let's Encrypt SSL certificates

## 🚀 Features

- Network-wide ad blocking
- Recursive DNS resolution (no reliance on third-party DNS)
- HTTPS access to Pi-hole admin interface
- Automatic SSL certificate generation and renewal
- Docker-based deployment

## 📋 Prerequisites

- Docker and Docker Compose installed
- Domain name configured in DNS (Cloudflare)
- Server with ports 53, 80, and 443 accessible from the internet

## 🛠️ Setup

### 1. Clone and Configure

```bash
git clone <your-repo>
cd oriongate
```

**Before deploying, update the following in `docker-compose.yml`:**

1. **Domain name** (3 locations):
   - Line 21: `VIRTUAL_HOST: pihole.yourdomain.com`
   - Line 35: `traefik.http.routers.pihole.rule=Host(\`pihole.yourdomain.com\`)`
   - Line 40: `traefik.http.routers.pihole-http.rule=Host(\`pihole.yourdomain.com\`)`

2. **Email address** (1 location):
   - Line 60: `--certificatesresolvers.le.acme.email=your-email@example.com`

3. **Timezone** (optional):
   - Line 19: `TZ: "America/Los_Angeles"`

### 2. DNS Configuration (Cloudflare)

1. Log into Cloudflare
2. Go to DNS settings for `yourdomain.com`
3. Add an A record:
   - **Type**: A
   - **Name**: `pihole`
   - **IPv4 address**: Your server's public IP
   - **Proxy status**: 🔴 **DNS only** (gray cloud - very important!)

### 3. Deploy

```bash
# Start all services
docker-compose up -d

# Check logs
docker-compose logs -f

# Check Traefik logs specifically for SSL certificate generation
docker-compose logs traefik
```

### 4. Verify Deployment

1. **DNS Resolution**: `nslookup pihole.yourdomain.com`
2. **SSL Certificate**: Visit `https://pihole.yourdomain.com/admin`
3. **DNS Service**: Configure a device to use your server's IP as DNS

## 🔧 Configuration

### Environment Variables

The setup uses these key configurations:

- **Timezone**: `America/Los_Angeles`
- **Domain**: `pihole.yourdomain.com`
- **Email**: `your-email@example.com` (for Let's Encrypt notifications)

### Volume Mounts

```
./pihole/etc-pihole/     # Pi-hole configuration
./pihole/etc-dnsmasq.d/  # DNS configuration
./traefik/letsencrypt/   # SSL certificates
```

## 🌐 Access Points

- **Pi-hole Admin**: `https://pihole.yourdomain.com/admin`
- **DNS Service**: Point devices to your server's IP address
- **Traefik Dashboard**: `http://your-server-ip:8080` (if exposed)

## 🔍 Troubleshooting

### SSL Certificate Issues

```bash
# Check Traefik logs
docker-compose logs traefik

# Verify domain resolution
nslookup pihole.yourdomain.com

# Check if ports are accessible
curl -I http://pihole.yourdomain.com
```

### DNS Resolution Issues

```bash
# Test DNS resolution
dig @your-server-ip google.com

# Check Unbound logs
docker-compose logs unbound

# Check Pi-hole logs
docker-compose logs pihole
```

### Common Issues

1. **SSL Certificate Fails**:
   - Ensure domain points to correct IP
   - Verify Cloudflare proxy is OFF (DNS only)
   - Check ports 80/443 are accessible

2. **Pi-hole Not Accessible**:
   - Check Traefik logs for routing issues
   - Verify networks are properly configured
   - Ensure Pi-hole is connected to frontend network

3. **DNS Not Working**:
   - Verify Unbound is running
   - Check Pi-hole upstream DNS configuration
   - Test with `dig @server-ip domain.com`

## 🔄 Updates

```bash
# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d
```

## 📁 Directory Structure

```
oriongate/
├── docker-compose.yml
├── README.md
├── pihole/
│   ├── etc-pihole/          # Created automatically
│   └── etc-dnsmasq.d/       # Created automatically
└── traefik/
    └── letsencrypt/         # Created automatically
```

## 🔒 Security Notes

- Pi-hole admin interface is only accessible via HTTPS
- Unbound provides recursive DNS resolution (no third-party DNS queries)
- Traefik handles SSL termination with automatic certificate renewal
- Services are isolated using Docker networks

## 📝 Customization

### Changing the Domain

1. Update the following 4 locations in `docker-compose.yml`:
   - Pi-hole VIRTUAL_HOST environment variable
   - Two Traefik router rules (HTTPS and HTTP redirect)
   - Traefik ACME email address

2. Update DNS records in your DNS provider (e.g., Cloudflare)

3. Restart services: `docker-compose up -d`

### Adding More Services

Add additional services to the `frontend` network to enable SSL access through Traefik.

## 📞 Support

For issues:
1. Check logs: `docker-compose logs`
2. Verify DNS configuration
3. Ensure firewall allows required ports
4. Check domain resolution and SSL certificate status
