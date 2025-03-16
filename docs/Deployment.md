# Deployment Guide

This guide provides step-by-step instructions to deploy the Domain Adaptation API on a VPS (Virtual Private Server) using either Nginx or Apache as a reverse proxy.

## Table of Contents

- [VPS Setup](#vps-setup)
- [Project Deployment](#project-deployment)
- [Nginx Configuration](#nginx-configuration)
- [Apache Configuration](#apache-configuration)
- [Systemd Service Setup](#systemd-service-setup)
- [SSL/TLS Configuration](#ssltls-configuration)
- [Troubleshooting](#troubleshooting)

## VPS Setup

### Requirements

- VPS with at least 4GB RAM (8GB+ recommended for larger models)
- Ubuntu 20.04 LTS or later
- Root access or sudo privileges

### Initial Server Setup

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install required dependencies
sudo apt install -y python3-pip python3-venv git nginx supervisor
```

### Setup Python Environment

```bash
# Install Python dependencies
sudo apt install -y python3-dev build-essential
```

## Project Deployment

### Clone Repository

```bash
# Create directory for application
mkdir -p /var/www/domain-api
cd /var/www/domain-api

# Clone the repository
git clone https://github.com/yourusername/domain-adaptation-api.git .
```

### Setup Virtual Environment

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
pip install gunicorn
```

### Test Application

```bash
# Test the application
uvicorn api.simple_api:app --host 127.0.0.1 --port 8000
```

If everything works, you should be able to access the API locally at http://127.0.0.1:8000.

## Nginx Configuration

### Install Nginx

```bash
sudo apt install -y nginx
```

### Create Nginx Configuration

Create a new configuration file:

```bash
sudo nano /etc/nginx/sites-available/domain-api
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com;  # Replace with your domain or server IP

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeout settings
        proxy_connect_timeout 75s;
        proxy_read_timeout 300s;
        client_max_body_size 50M;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/domain-api /etc/nginx/sites-enabled/
sudo nginx -t  # Test configuration
sudo systemctl restart nginx
```

## Apache Configuration

### Install Apache

```bash
sudo apt install -y apache2
```

### Enable Required Modules

```bash
sudo a2enmod proxy proxy_http headers ssl rewrite
sudo systemctl restart apache2
```

### Create Apache Configuration

Create a new configuration file:

```bash
sudo nano /etc/apache2/sites-available/domain-api.conf
```

Add the following configuration:

```apache
<VirtualHost *:80>
    ServerName your-domain.com  # Replace with your domain or server IP
    
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
    
    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>
    
    ErrorLog ${APACHE_LOG_DIR}/domain-api-error.log
    CustomLog ${APACHE_LOG_DIR}/domain-api-access.log combined
</VirtualHost>
```

Enable the configuration:

```bash
sudo a2ensite domain-api
sudo apache2ctl configtest  # Test configuration
sudo systemctl restart apache2
```

## Systemd Service Setup

Create a systemd service file:

```bash
sudo nano /etc/systemd/system/domain-api.service
```

Add the following configuration:

```ini
[Unit]
Description=Domain Adaptation API Service
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/domain-api
ExecStart=/var/www/domain-api/venv/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker -b 127.0.0.1:8000 api.simple_api:app
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=domain-api
Environment="PATH=/var/www/domain-api/venv/bin"

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable domain-api
sudo systemctl start domain-api
sudo systemctl status domain-api
```

## SSL/TLS Configuration

### Using Certbot with Nginx

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

### Using Certbot with Apache

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d your-domain.com
```

## Performance Tuning

### Gunicorn Workers

Adjust the number of workers in the systemd service file based on your server's CPU cores:

```
# General rule: (2 x CPU cores) + 1
ExecStart=/var/www/domain-api/venv/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker -b 127.0.0.1:8000 api.simple_api:app
```

### Memory Considerations

For larger models, you may need to increase the swap space:

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Troubleshooting

### Checking Logs

```bash
# Check API service logs
sudo journalctl -u domain-api

# Check Nginx logs
sudo tail -f /var/log/nginx/error.log

# Check Apache logs
sudo tail -f /var/log/apache2/domain-api-error.log
```

### Common Issues

1. **Permission Errors**:
   ```bash
   sudo chown -R www-data:www-data /var/www/domain-api
   sudo chmod -R 755 /var/www/domain-api
   ```

2. **Firewall Configuration**:
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```

3. **Model Loading Failures**:
   - Check if the model path exists
   - Ensure the service user has read permissions for the model directory
   - Check for sufficient memory
