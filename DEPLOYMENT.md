# 🚀 Production Deployment Guide

Complete guide for deploying IONOS AI Model Hub + n8n integration to your production environment.

## 📋 Current Production Setup Analysis

### Current Configuration
- **Domain**: https://360foresight.info
- **n8n Version**: 1.112.4
- **SSL**: Let's Encrypt (Certbot managed)
- **Reverse Proxy**: Nginx
- **Container**: Docker with persistent volume `n8n_data`
- **IONOS Integration**: ✅ Already configured

### Current Architecture
```
Internet → Nginx (443/SSL) → Docker Container (n8n:5678) → Volume (n8n_data)
```

## 🔧 Production Update Steps

### Step 1: Backup Current Installation

```bash
# 1. Backup n8n data volume
docker run --rm -v n8n_data:/data -v /root/backup:/backup alpine \
    tar czf /backup/n8n_backup_$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# 2. Export current workflows
docker exec n8n n8n export:workflow --all --output=/tmp/workflows_backup_$(date +%Y%m%d).json

# 3. Copy backup from container
docker cp n8n:/tmp/workflows_backup_$(date +%Y%m%d).json /root/backup/

# 4. Backup nginx config
cp -r /etc/nginx/sites-enabled /root/backup/nginx_sites_$(date +%Y%m%d)/
cp -r /etc/letsencrypt /root/backup/letsencrypt_$(date +%Y%m%d)/

# 5. Document current environment
docker inspect n8n > /root/backup/n8n_container_config_$(date +%Y%m%d).json
```

### Step 2: Update Production Container

```bash
# 1. Pull latest n8n image
docker pull n8nio/n8n:latest

# 2. Stop current container (minimal downtime)
docker stop n8n

# 3. Remove old container (keep volume)
docker rm n8n

# 4. Start new container with IONOS integration
docker run -d --name n8n \
    --restart unless-stopped \
    -p 5678:5678 \
    -v n8n_data:/home/node/.n8n \
    -e IONOS_API_KEY="$IONOS_API_KEY" \
    -e N8N_HOST="360foresight.info" \
    -e N8N_PORT=5678 \
    -e N8N_PROTOCOL=https \
    -e WEBHOOK_URL="https://360foresight.info/" \
    -e GENERIC_TIMEZONE="Europe/Berlin" \
    -e TZ="Europe/Berlin" \
    n8nio/n8n:latest

# 5. Verify container is running
docker ps | grep n8n
docker logs n8n --tail 20

# 6. Test IONOS integration
docker exec n8n curl -s -H "Authorization: Bearer $IONOS_API_KEY" \
    https://openai.inference.de-txl.ionos.com/v1/models | head -5
```

### Step 3: Install IONOS Community Node

```bash
# Install the community node
docker exec n8n npm install @ionos-cloud/n8n-nodes-ionos-cloud

# Restart container to load new nodes
docker restart n8n

# Wait for startup
sleep 30

# Verify installation
docker exec n8n npm list | grep ionos
```

### Step 4: Import Example Workflows

```bash
# 1. Copy workflow files to container
docker cp /root/ionos-n8n-ai-hub/workflows/ n8n:/tmp/ionos-workflows/

# 2. Import workflows (done via n8n UI or API)
# Navigate to https://360foresight.info
# Settings → Import from File → Select workflow JSON files
```

### Step 5: Verification

```bash
# 1. Check container health
docker exec n8n n8n --version
docker exec n8n printenv | grep IONOS

# 2. Test domain access
curl -I https://360foresight.info

# 3. Test IONOS API
curl -H "Authorization: Bearer $IONOS_API_KEY" \
    https://openai.inference.de-txl.ionos.com/v1/models

# 4. Check SSL certificate
openssl s_client -connect 360foresight.info:443 -servername 360foresight.info < /dev/null | grep -A2 "Verify return code"
```

## 🔄 Rollback Plan (If Something Goes Wrong)

```bash
# 1. Stop new container
docker stop n8n && docker rm n8n

# 2. Restore from backup
docker run --rm -v n8n_data:/data -v /root/backup:/backup alpine \
    tar xzf /backup/n8n_backup_YYYYMMDD_HHMMSS.tar.gz -C /data

# 3. Start original container
docker run -d --name n8n \
    --restart unless-stopped \
    -p 5678:5678 \
    -v n8n_data:/home/node/.n8n \
    n8nio/n8n:1.112.4  # Use specific version

# 4. Import workflows if needed
docker exec n8n n8n import:workflow --input=/tmp/workflows_backup_YYYYMMDD.json
```

## 📱 Mobile-Friendly Management

### Quick Commands via SSH
```bash
# Check status
alias n8n-status='docker ps | grep n8n && docker logs n8n --tail 10'

# Restart n8n
alias n8n-restart='docker restart n8n && sleep 10 && docker logs n8n --tail 5'

# Check IONOS API
alias ionos-test='docker exec n8n curl -s -H "Authorization: Bearer $IONOS_API_KEY" https://openai.inference.de-txl.ionos.com/v1/models | head -3'

# Backup workflows
alias n8n-backup='docker exec n8n n8n export:workflow --all --output=/tmp/backup_$(date +%Y%m%d_%H%M%S).json'
```

## 🛡️ Security Considerations

### Environment Variables
```bash
# Store IONOS key securely
echo 'export IONOS_API_KEY="your_key_here"' >> /root/.bashrc
chmod 600 /root/.bashrc

# Or use Docker secrets (recommended for production)
echo "your_key_here" | docker secret create ionos_api_key -
```

### Firewall Rules
```bash
# Ensure only necessary ports are open
ufw status
ufw allow 22    # SSH
ufw allow 80    # HTTP (redirects to HTTPS)
ufw allow 443   # HTTPS
ufw deny 5678   # Block direct n8n access (only via nginx)
```

### SSL Certificate Auto-Renewal
```bash
# Check certbot timer
systemctl status certbot.timer

# Test renewal
certbot renew --dry-run

# Manual renewal if needed
certbot renew
systemctl restart nginx
```

## 📊 Monitoring Setup

### Health Check Script
```bash
#!/bin/bash
# /root/scripts/health_check.sh

echo "🏥 Health Check - $(date)"

# Check Docker container
if docker ps | grep -q "n8n.*Up"; then
    echo "✅ n8n container running"
else
    echo "❌ n8n container not running"
    docker start n8n
fi

# Check domain access
if curl -s -I https://360foresight.info | grep -q "200 OK"; then
    echo "✅ Domain accessible"
else
    echo "❌ Domain not accessible"
fi

# Check IONOS API
if docker exec n8n curl -s -H "Authorization: Bearer $IONOS_API_KEY" \
    https://openai.inference.de-txl.ionos.com/v1/models | grep -q "data"; then
    echo "✅ IONOS API working"
else
    echo "❌ IONOS API not working"
fi

# Check disk space
df -h | grep -E "(/$|/var|/home)"
```

### Cron Jobs
```bash
# Add to crontab: crontab -e
# Health check every 15 minutes
*/15 * * * * /root/scripts/health_check.sh >> /var/log/n8n_health.log 2>&1

# Daily backup at 2 AM
0 2 * * * /root/scripts/backup_n8n.sh >> /var/log/n8n_backup.log 2>&1

# SSL renewal check (first day of month)
0 3 1 * * certbot renew --quiet && systemctl restart nginx
```

## 🔧 Maintenance Tasks

### Weekly Tasks
```bash
#!/bin/bash
# /root/scripts/weekly_maintenance.sh

# Update Docker images
docker pull n8nio/n8n:latest

# Clean up old images
docker image prune -f

# Backup workflows
docker exec n8n n8n export:workflow --all --output=/tmp/weekly_backup_$(date +%Y%m%d).json
docker cp n8n:/tmp/weekly_backup_$(date +%Y%m%d).json /root/backup/

# Check logs for errors
docker logs n8n --since="7 days" | grep -i error > /tmp/n8n_errors.log

# System updates (Ubuntu)
apt update && apt list --upgradable
```

### Monthly Tasks
```bash
# Full system backup
tar czf /root/backup/full_system_$(date +%Y%m%d).tar.gz \
    /etc/nginx/ /etc/letsencrypt/ /root/backup/ /root/scripts/

# Certificate renewal
certbot renew
systemctl restart nginx

# Docker cleanup
docker system prune -af --volumes
```

## 📞 Support Information

### Important Files Locations
```
/root/backup/                 # All backups
/root/scripts/               # Maintenance scripts
/etc/nginx/sites-enabled/    # Nginx configuration
/etc/letsencrypt/           # SSL certificates
/var/log/n8n_*.log         # Application logs
```

### Container Information
```
Container Name: n8n
Image: n8nio/n8n:latest
Volume: n8n_data
Ports: 5678 (internal only, accessed via nginx)
Domain: https://360foresight.info
```

### Emergency Contacts
- **Server Admin**: support@360foresight.info
- **IONOS Support**: [IONOS Cloud Dashboard](https://cloud.ionos.com)
- **Domain/SSL**: Let's Encrypt via Certbot

---

## 🔗 Git Repository Setup

### Repository Information
- **Local Path**: `/root/ionos-n8n-ai-hub/`
- **Remote**: To be created on GitHub
- **Documentation**: Complete with workflows and examples

### Next Session Setup Required
Please provide in next session:
1. **GitHub repository URL** (after creating it)
2. **Git access method** (SSH key, token, etc.)
3. **Any specific deployment preferences**

The repository is ready to push with:
- Complete documentation
- Working workflow examples
- Production deployment guide
- Best practices and troubleshooting

---

**🎯 Ready for Production!** Your n8n instance is now configured with IONOS AI Model Hub integration and ready for the community to learn from.