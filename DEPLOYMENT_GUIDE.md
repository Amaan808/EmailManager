# AI Communication Assistant - Deployment Guide

## 🚀 Production Deployment Guide

### Overview

This guide covers the complete deployment process for the AI Communication Assistant from development to production environment.

---

## 📋 Pre-Deployment Checklist

### Development Environment Verification

- [ ] All features working correctly in development
- [ ] Database migrations completed
- [ ] Environment variables configured
- [ ] Tests passing (if applicable)
- [ ] AI service (GROQ) operational
- [ ] Code committed to version control

### Production Requirements

- [ ] Node.js 18+ installed on server
- [ ] PostgreSQL 12+ database server
- [ ] SSL certificate for HTTPS
- [ ] Domain name configured
- [ ] Firewall rules configured
- [ ] Backup strategy implemented

---

## 🖥️ Server Setup

### Minimum System Requirements

#### Hardware:

- **CPU**: 2+ cores
- **RAM**: 4GB minimum, 8GB recommended
- **Storage**: 50GB SSD minimum
- **Network**: 100Mbps+ bandwidth

#### Software:

- **OS**: Ubuntu 20.04 LTS or CentOS 8+
- **Node.js**: v18.x or higher
- **PostgreSQL**: v12 or higher
- **Nginx**: v1.18+ (for reverse proxy)
- **PM2**: Process manager for Node.js

### Initial Server Setup

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Install Nginx
sudo apt install nginx

# Install PM2 globally
sudo npm install -g pm2

# Install Git
sudo apt install git
```

### PostgreSQL Database Setup

```bash
# Switch to postgres user
sudo -u postgres psql

# Create database and user
CREATE DATABASE ai_communication_db;
CREATE USER ai_app_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE ai_communication_db TO ai_app_user;
\q

# Configure PostgreSQL for remote connections (if needed)
sudo nano /etc/postgresql/12/main/postgresql.conf
# Uncomment and modify: listen_addresses = '*'

sudo nano /etc/postgresql/12/main/pg_hba.conf
# Add: host ai_communication_db ai_app_user 0.0.0.0/0 md5

# Restart PostgreSQL
sudo systemctl restart postgresql
```

---

## 📦 Application Deployment

### 1. Clone and Setup Application

```bash
# Clone the repository
cd /var/www/
sudo git clone [your-repository-url] ai-communication-assistant
sudo chown -R $USER:$USER ai-communication-assistant
cd ai-communication-assistant

# Install backend dependencies
cd backend
npm install --production

# Install frontend dependencies
cd ../frontend
npm install

# Build frontend for production
npm run build

# Copy build files to serve directory
sudo cp -r build/* /var/www/html/
```

### 2. Environment Configuration

Create production environment file:

```bash
# Backend environment
cd /var/www/ai-communication-assistant/backend
nano .env
```

**Production .env file:**

```env
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=ai_communication_db
DB_USER=ai_app_user
DB_PASSWORD=your_secure_password

# Server Configuration
PORT=5001
NODE_ENV=production

# GROQ AI Configuration
GROQ_API_KEY=your_groq_api_key_here

# Security
JWT_SECRET=your_jwt_secret_here
SESSION_SECRET=your_session_secret_here

# CORS Configuration
FRONTEND_URL=https://yourdomain.com

# Email Configuration (if needed)
SMTP_HOST=your.smtp.server.com
SMTP_PORT=587
SMTP_USER=your_email@yourdomain.com
SMTP_PASS=your_email_password

# Logging
LOG_LEVEL=info
LOG_FILE=/var/log/ai-communication-assistant/app.log
```

### 3. Database Migration

```bash
cd /var/www/ai-communication-assistant/backend

# Run database setup script
node scripts/setup-database.js

# Import initial data (if needed)
node scripts/import-csv.js
```

### 4. PM2 Process Management

Create PM2 ecosystem file:

```bash
cd /var/www/ai-communication-assistant
nano ecosystem.config.js
```

**ecosystem.config.js:**

```javascript
module.exports = {
  apps: [
    {
      name: "ai-communication-backend",
      script: "./backend/server.js",
      cwd: "/var/www/ai-communication-assistant",
      instances: "max",
      exec_mode: "cluster",
      env: {
        NODE_ENV: "production",
        PORT: 5001,
      },
      log_file: "/var/log/ai-communication-assistant/combined.log",
      out_file: "/var/log/ai-communication-assistant/out.log",
      error_file: "/var/log/ai-communication-assistant/error.log",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      max_memory_restart: "1G",
      watch: false,
      autorestart: true,
    },
  ],
};
```

### 5. Start Application with PM2

```bash
# Create log directory
sudo mkdir -p /var/log/ai-communication-assistant
sudo chown $USER:$USER /var/log/ai-communication-assistant

# Start application
cd /var/www/ai-communication-assistant
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 to start on boot
pm2 startup
# Follow the instructions provided by the command above
```

---

## 🌐 Nginx Configuration

### SSL Certificate Setup (Let's Encrypt)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

### Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/ai-communication-assistant
```

**Nginx configuration:**

```nginx
# HTTP redirect to HTTPS
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS configuration
server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # SSL Security Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Root directory for frontend
    root /var/www/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    # Frontend routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy to backend
    location /api/ {
        proxy_pass http://localhost:5001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 300s;
        proxy_connect_timeout 75s;
    }

    # Static files caching
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Log files
    access_log /var/log/nginx/ai-communication-assistant.access.log;
    error_log /var/log/nginx/ai-communication-assistant.error.log;
}
```

### Enable and Test Nginx

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/ai-communication-assistant /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx

# Enable Nginx to start on boot
sudo systemctl enable nginx
```

---

## 🔐 Security Configuration

### Firewall Setup

```bash
# Install UFW
sudo apt install ufw

# Configure firewall rules
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

### Application Security

1. **Environment Variables**: Never commit sensitive data to version control
2. **Database Security**: Use strong passwords and limit database access
3. **API Security**: Implement rate limiting and input validation
4. **HTTPS Only**: Force all traffic through HTTPS
5. **Regular Updates**: Keep all system packages updated

### Backup Strategy

```bash
# Create backup script
sudo nano /usr/local/bin/backup-ai-app.sh
```

**Backup script:**

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/ai-communication-assistant"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="ai_communication_db"
DB_USER="ai_app_user"

# Create backup directory
mkdir -p $BACKUP_DIR

# Database backup
pg_dump -h localhost -U $DB_USER $DB_NAME | gzip > $BACKUP_DIR/database_$DATE.sql.gz

# Application files backup
tar -czf $BACKUP_DIR/application_$DATE.tar.gz /var/www/ai-communication-assistant

# Remove backups older than 30 days
find $BACKUP_DIR -type f -mtime +30 -delete

echo "Backup completed: $DATE"
```

```bash
# Make script executable
sudo chmod +x /usr/local/bin/backup-ai-app.sh

# Add to crontab for daily backups
sudo crontab -e
# Add: 0 2 * * * /usr/local/bin/backup-ai-app.sh
```

---

## 📊 Monitoring & Logging

### Application Monitoring

```bash
# Monitor PM2 processes
pm2 status
pm2 logs
pm2 monit

# Monitor system resources
htop
df -h
free -h
```

### Log Management

```bash
# Setup log rotation
sudo nano /etc/logrotate.d/ai-communication-assistant
```

**Log rotation config:**

```
/var/log/ai-communication-assistant/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 www-data www-data
    postrotate
        pm2 reload ai-communication-backend
    endscript
}
```

### Health Monitoring Script

```bash
sudo nano /usr/local/bin/health-check.sh
```

**Health check script:**

```bash
#!/bin/bash
API_URL="https://yourdomain.com/api/health"
RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" $API_URL)

if [ $RESPONSE -eq 200 ]; then
    echo "$(date): Application is healthy"
else
    echo "$(date): Application health check failed (HTTP $RESPONSE)"
    # Optional: restart application
    # pm2 restart ai-communication-backend
fi
```

---

## 🚦 Testing Deployment

### Deployment Verification Checklist

#### Frontend Tests:

- [ ] Website loads correctly at your domain
- [ ] HTTPS certificate is valid
- [ ] All pages render properly
- [ ] Navigation works correctly
- [ ] No console errors in browser

#### Backend Tests:

- [ ] API endpoints respond correctly
- [ ] Database connection is working
- [ ] GROQ AI service is operational
- [ ] Error handling works properly
- [ ] Logs are being generated

#### Performance Tests:

- [ ] Page load times are acceptable (<3 seconds)
- [ ] API response times are fast (<1 second)
- [ ] System handles concurrent users
- [ ] Memory and CPU usage are normal

### Manual Testing Workflow

1. **Access the application** at your domain
2. **Import test data** using CSV import
3. **Process emails** with AI features
4. **Generate responses** and verify quality
5. **Check analytics** dashboard for accuracy
6. **Test error scenarios** (network issues, invalid data)

---

## 🔄 Maintenance & Updates

### Regular Maintenance Tasks

#### Daily:

- [ ] Check application logs for errors
- [ ] Monitor system resources
- [ ] Verify backup completion

#### Weekly:

- [ ] Review application performance
- [ ] Update system packages
- [ ] Check SSL certificate expiration

#### Monthly:

- [ ] Review and optimize database
- [ ] Update application dependencies
- [ ] Test backup restoration process

### Update Deployment Process

```bash
# 1. Backup current version
/usr/local/bin/backup-ai-app.sh

# 2. Pull latest code
cd /var/www/ai-communication-assistant
git pull origin main

# 3. Update dependencies
cd backend && npm install --production
cd ../frontend && npm install && npm run build

# 4. Copy new build files
sudo cp -r frontend/build/* /var/www/html/

# 5. Restart application
pm2 restart ai-communication-backend

# 6. Verify deployment
curl https://yourdomain.com/api/health
```

---

## 🆘 Troubleshooting

### Common Issues

#### Application Won't Start:

- Check PM2 logs: `pm2 logs`
- Verify environment variables
- Check database connection
- Ensure all dependencies are installed

#### Database Connection Issues:

- Verify PostgreSQL is running: `sudo systemctl status postgresql`
- Check database credentials
- Test connection manually
- Review firewall settings

#### High Memory Usage:

- Check for memory leaks in logs
- Restart application: `pm2 restart ai-communication-backend`
- Monitor with: `pm2 monit`
- Consider increasing server RAM

#### SSL Certificate Issues:

- Renew certificate: `sudo certbot renew`
- Check certificate expiration: `sudo certbot certificates`
- Verify Nginx configuration: `sudo nginx -t`

### Emergency Recovery

#### If Application is Down:

1. **Check PM2 status**: `pm2 status`
2. **Restart application**: `pm2 restart ai-communication-backend`
3. **Check logs**: `pm2 logs`
4. **Verify database**: `sudo systemctl status postgresql`
5. **Contact support** if issues persist

#### If Database is Corrupted:

1. **Stop application**: `pm2 stop ai-communication-backend`
2. **Restore from backup**: Use latest database backup
3. **Restart services**: `sudo systemctl restart postgresql`
4. **Start application**: `pm2 start ai-communication-backend`

---

## 📞 Support & Maintenance

### Production Support Contacts

- **System Administrator**: [Your Admin Contact]
- **Database Administrator**: [Your DBA Contact]
- **Application Developer**: [Your Dev Contact]
- **Emergency Contact**: [Your Emergency Contact]

### Monitoring Dashboard URLs

- **Application**: https://yourdomain.com
- **PM2 Monitor**: `pm2 monit` (server access required)
- **System Logs**: `/var/log/ai-communication-assistant/`
- **Nginx Logs**: `/var/log/nginx/`

---

**Deployment Guide Version:** 1.0  
**Last Updated:** September 6, 2025  
**Target Environment:** Production  
**Status:** Ready for Deployment
