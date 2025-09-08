---
title: "Networking"
description: "Ports, TLS, proxies, headers, timeouts configuration."
tags: [networking, security, configuration]
search: { boost: 3, exclude: false }
icon: material/network
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document network configuration and connectivity requirements.

**Contents**
- [Port Configuration](#port-configuration)
- [Proxy & Load Balancer](#proxy--load-balancer)
- [TLS & Security](#tls--security)
- [Container Networking](#container-networking)
- [Sources](#sources)

## Port Configuration

**Container Ports:**

| Service | Internal Port | External Port | Protocol | Purpose | Source |
|---------|---------------|---------------|----------|---------|--------|
| Nginx | 80 | 8080 | HTTP | Web interface | "Docker Compose Config" — docker-compose.yml — retrieved 2025-01-09 |
| PHP-FPM | 9000 | 9000 | FastCGI | Application backend | "Docker Compose Config" — docker-compose.yml — retrieved 2025-01-09 |
| MariaDB | 3306 | 3306 | MySQL | Database access | "Docker Compose Config" — docker-compose.yml — retrieved 2025-01-09 |
| Redis | 6379 | N/A | Redis | Cache/sessions (internal only) | "Docker Compose Config" — docker-compose.yml — retrieved 2025-01-09 |

**Port Binding Security:**
- All ports bound to `127.0.0.1` (localhost only)
- No direct external access to database or cache
- Web traffic routed through nginx proxy
- PHP-FPM exposed for direct debugging only

**Firewall Configuration:**
```bash
# Allow HTTP traffic
sudo ufw allow 8080/tcp

# Allow HTTPS traffic (if using external TLS termination)
sudo ufw allow 443/tcp

# Block direct database access from external networks
sudo ufw deny 3306/tcp
```

## Proxy & Load Balancer

**Nginx Configuration:**
```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/app/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass firefly-iii-application.application:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # Performance tuning
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_read_timeout 300;
    }
}
```

**External Load Balancer (Production):**
```yaml
# Example Ingress configuration
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: firefly-iii-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "64m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
spec:
  rules:
  - host: firefly.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: firefly-iii-service
            port:
              number: 80
```

**Trusted Proxies:**
```bash
# Environment variable for proxy trust
TRUSTED_PROXIES=*

# Or specific proxy IPs
TRUSTED_PROXIES=10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```

## TLS & Security

**TLS Termination Options:**

| Option | Implementation | Pros | Cons | Source |
|--------|----------------|------|------|--------|
| External Load Balancer | Kubernetes Ingress, AWS ALB | Centralized, automatic renewal | Additional infrastructure | "Firefly III Security" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Nginx Proxy | Reverse proxy with Let's Encrypt | Simple setup, full control | Manual certificate management | "Firefly III Security" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Container TLS | Direct certificate in nginx container | No external dependencies | Complex certificate rotation | "Firefly III Security" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Security Headers:**
```nginx
# Add to nginx configuration
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy strict-origin-when-cross-origin;
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'";
```

**SSL/TLS Configuration:**
```nginx
# Strong SSL configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;

# HSTS header
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

## Container Networking

**Docker Compose Network:**
```yaml
networks:
  default:
    external: true
    name: webgrip
```

**Container Communication:**
- Containers communicate via service names
- Internal DNS resolution provided by Docker
- No network isolation between services (shared network)
- External network allows integration with other services

**Network Security:**
```bash
# Check container network connectivity
docker exec firefly-iii-application.application nc -zv firefly-iii-application.mariadb 3306
docker exec firefly-iii-application.application nc -zv firefly-iii-application.redis 6379

# Monitor network traffic (debugging)
docker run --rm --net container:firefly-iii-application.application nicolaka/netshoot tcpdump -i eth0
```

**Service Discovery:**
- MariaDB: `firefly-iii-application.mariadb:3306`
- Redis: `firefly-iii-application.redis:6379`
- Application: `firefly-iii-application.application:9000`
- Nginx: `firefly-iii-application.nginx:80`

## Performance & Timeouts

**Connection Timeouts:**
```bash
# PHP-FPM settings
max_execution_time=300
max_input_time=300
memory_limit=512M

# Nginx FastCGI timeouts
fastcgi_connect_timeout=60s
fastcgi_send_timeout=300s
fastcgi_read_timeout=300s
```

**Database Connection Pool:**
```bash
# MariaDB connection settings
max_connections=151
connect_timeout=10
wait_timeout=28800
interactive_timeout=28800
```

**Redis Network Configuration:**
```bash
# Redis timeout settings
timeout=0
tcp-keepalive=300
tcp-backlog=511
```

## Monitoring & Diagnostics

**Network Health Checks:**
```bash
# Test all service connectivity
curl -f http://localhost:8080/health
telnet localhost 3306
redis-cli -h localhost ping

# Check DNS resolution
docker exec firefly-iii-application.application nslookup firefly-iii-application.mariadb
```

**Traffic Analysis:**
```bash
# Monitor HTTP requests
docker logs firefly-iii-application.nginx | grep "GET\|POST"

# Database connection monitoring
docker exec firefly-iii-application.mariadb mysql -u root -p -e "SHOW PROCESSLIST;"

# Redis connection monitoring
docker exec firefly-iii-application.redis redis-cli info clients
```

**Troubleshooting Network Issues:**

1. **Connection Refused:**
   - Check service is running: `docker ps`
   - Verify port binding: `netstat -tlnp | grep :8080`
   - Check firewall rules: `sudo ufw status`

2. **Slow Response Times:**
   - Monitor container resources: `docker stats`
   - Check database performance: `SHOW PROCESSLIST`
   - Analyze nginx access logs for slow requests

3. **DNS Resolution Issues:**
   - Verify container network: `docker network inspect webgrip`
   - Test internal connectivity: `docker exec ... nc -zv service_name port`

## Sources
- "Firefly III Installation Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09
- "Docker Compose Configuration" — docker-compose.yml — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":"","docker-compose.yml":""},"sections":{"networking":""}}}
-->