# Security

**Purpose**: This document outlines security considerations, best practices, and procedures for securing the Firefly III application in development and production environments.

## Table of Contents

- [Security Overview](#security-overview)
- [Authentication and Authorization](#authentication-and-authorization)
- [Data Protection](#data-protection)
- [Network Security](#network-security)
- [Container Security](#container-security)
- [Secrets Management](#secrets-management)
- [Security Monitoring](#security-monitoring)
- [Compliance Considerations](#compliance-considerations)

## Security Overview

### Security Principles

This Firefly III deployment follows industry security best practices:
- **Defense in Depth**: Multiple layers of security controls
- **Principle of Least Privilege**: Minimal necessary permissions
- **Secure by Default**: Security-focused default configurations
- **Zero Trust**: Verify everything, trust nothing
- **Data Minimization**: Collect and store only necessary data

### Threat Model

**Primary Threats**:
- Unauthorized access to financial data
- Data exfiltration or corruption
- Service disruption or availability attacks
- Privilege escalation within containers
- Man-in-the-middle attacks

**Attack Vectors**:
- Web application vulnerabilities
- Container escape attempts
- Database injection attacks
- Network interception
- Social engineering

## Authentication and Authorization

### Application Authentication

**Default Authentication**:
- Email and password-based login
- Session-based authentication via Laravel
- CSRF protection on all forms
- Password strength requirements

**Security Configuration**:
```env
# Strong session security
SESSION_SECURE=true          # HTTPS only in production
SESSION_ENCRYPT=true         # Encrypt session data
SESSION_DOMAIN=yourdomain.com # Restrict to specific domain

# Password requirements (configured in Firefly III)
# - Minimum 12 characters
# - Mixed case letters
# - Numbers and special characters
```

**Multi-Factor Authentication (MFA)**:
Firefly III supports optional MFA via:
- Google Authenticator
- Authy
- Any TOTP-compatible app

Enable MFA in production:
```bash
# Enable MFA for all users
make run CMD="php artisan firefly-iii:enable-mfa"
```

### API Authentication

**API Security**:
- Personal Access Tokens for API access
- Rate limiting on API endpoints
- Scope-based permissions
- Token expiration policies

**API Token Management**:
```bash
# Generate API token for user
make run CMD="php artisan passport:client --personal"

# Revoke tokens
make run CMD="php artisan passport:purge"
```

### Administrative Access

**Container Access**:
```bash
# Use make commands for container access
make enter                    # Shell access
make run CMD="command"       # Execute commands

# Avoid direct docker exec for security auditing
```

**Database Access**:
```bash
# Use application database user (limited permissions)
# Avoid root access in production

# Emergency root access (log all actions)
docker compose exec firefly-iii-application.mariadb \
  mysql -u root -p${DB_ROOT_PASSWORD}
```

## Data Protection

### Data Encryption

**Encryption at Rest**:
- Database volume encryption (filesystem level)
- Application secrets encrypted via Laravel
- Backup encryption using SOPS/age

**Encryption in Transit**:
- HTTPS for all external communication
- TLS for database connections (optional)
- Internal container communication over encrypted networks

**Encryption Configuration**:
```env
# Application encryption key (required)
APP_KEY=base64:...

# Database connection encryption (optional)
DB_ENCRYPT=true
DB_SSL_CA=/path/to/ca.pem
DB_SSL_CERT=/path/to/cert.pem
DB_SSL_KEY=/path/to/key.pem
```

### Data Classification

**Financial Data** (Highly Sensitive):
- Transaction records
- Account balances
- Budget information
- Investment data

**Personal Data** (Sensitive):
- User profiles
- Email addresses
- Personal preferences
- Authentication data

**Application Data** (Internal):
- Logs (sanitized)
- System configuration
- Application metrics

### Data Retention

**Retention Policies**:
- **Active Data**: Retained as long as account is active
- **Deleted Accounts**: Data purged after 30 days
- **Logs**: Retained for 90 days maximum
- **Backups**: Follow organizational policy

**Data Purging**:
```bash
# Purge deleted users (after retention period)
make run CMD="php artisan firefly-iii:purge-deleted-users"

# Clear old logs
make run CMD="php artisan log:clear --days=90"
```

## Network Security

### Container Networking

**Network Isolation**:
- Services communicate via internal Docker network
- No direct external access to database or cache
- Nginx as single point of entry

**Port Security**:
```yaml
# docker-compose.yml security configuration
ports:
  - "127.0.0.1:8080:80"  # Bind to localhost only
  - "127.0.0.1:3306:3306"  # Database (dev only, remove in production)
```

**Firewall Configuration**:
```bash
# Production firewall rules
sudo ufw deny 3306/tcp         # Block direct database access
sudo ufw allow 80/tcp          # HTTP
sudo ufw allow 443/tcp         # HTTPS
sudo ufw allow 22/tcp          # SSH (restrict source IPs)
```

### HTTPS/TLS Configuration

**SSL/TLS Setup** (Production):
```nginx
# Nginx TLS configuration
server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    ssl_certificate /etc/ssl/certs/your-domain.crt;
    ssl_certificate_key /etc/ssl/private/your-domain.key;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

**Certificate Management**:
```bash
# Using Let's Encrypt with Certbot
certbot --nginx -d your-domain.com

# Automatic renewal
echo "0 3 * * * /usr/bin/certbot renew --quiet" | crontab -
```

## Container Security

### Container Hardening

**Security Features**:
- Non-root user execution
- Read-only filesystems where possible
- Resource limits to prevent DoS
- Security profiles (AppArmor/SELinux)

**Container Configuration**:
```yaml
# docker-compose.yml security settings
services:
  firefly-iii-application.application:
    user: "www-data:www-data"
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
```

### Image Security

**Image Scanning**:
```bash
# Scan images for vulnerabilities
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image webgrip/firefly-iii-application:latest

# Scan for secrets
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  trufflesecurity/trufflehog:latest docker \
  --image webgrip/firefly-iii-application:latest
```

**Image Verification**:
```bash
# Verify image signatures (if implemented)
docker trust inspect webgrip/firefly-iii-application:latest

# Check image layers
docker history webgrip/firefly-iii-application:latest
```

### Runtime Security

**Runtime Monitoring**:
```bash
# Monitor container behavior
docker events --filter container=firefly-iii-application.application

# Check container processes
docker exec firefly-iii-application.application ps aux

# Monitor file changes
docker diff firefly-iii-application.application
```

## Secrets Management

### Environment Variables

**Secure Environment Configuration**:
```env
# Use strong, unique secrets
APP_KEY=base64:$(openssl rand -base64 32)
API_SECRET=$(openssl rand -hex 32)
UPDATE_SECRET=$(openssl rand -hex 32)
WEBCRON_SECRET=$(openssl rand -hex 16)

# Database credentials
DB_PASSWORD=$(openssl rand -base64 24)
DB_ROOT_PASSWORD=$(openssl rand -base64 32)
```

**Secret Rotation**:
```bash
# Rotate application key (requires application restart)
make run CMD="php artisan key:generate"

# Rotate database passwords
# 1. Update password in database
# 2. Update .env file
# 3. Restart application
```

### SOPS Integration

**Encrypted Secrets Management**:
```bash
# Initialize SOPS encryption
make secrets:init

# Encrypt secrets file
make secrets:encrypt SECRETS_DIR=./ops/secrets/firefly-iii-application-secrets

# Decrypt for deployment
make secrets:decrypt SECRETS_DIR=./ops/secrets/firefly-iii-application-secrets
```

**SOPS Configuration**:
```yaml
# .sops.yaml
creation_rules:
  - path_regex: \.dec\.yaml$
    age: age1...your_public_key
    encrypted_regex: '^(password|secret|key|token)$'
```

### External Secrets

**Production Secrets Management**:
- Use external secret management (HashiCorp Vault, AWS Secrets Manager)
- Implement secret rotation policies
- Audit secret access and usage
- Use short-lived tokens where possible

## Security Monitoring

### Log Monitoring

**Security Events to Monitor**:
- Failed login attempts
- Privilege escalation attempts
- Unusual data access patterns
- Container runtime anomalies
- Network connection attempts

**Log Analysis**:
```bash
# Monitor failed logins
docker compose logs firefly-iii-application.application | grep "Failed login"

# Monitor database connections
docker compose logs firefly-iii-application.mariadb | grep "Access denied"

# Monitor container events
docker events --filter type=container --filter event=die
```

**SIEM Integration**:
```bash
# Export logs for SIEM analysis
docker compose logs --no-color > /var/log/firefly-iii/application.log

# Forward to central logging
# Configure Fluentd, Logstash, or similar
```

### Vulnerability Scanning

**Regular Security Scans**:
```bash
# Weekly vulnerability scan
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --severity HIGH,CRITICAL \
  webgrip/firefly-iii-application:latest

# Check for exposed secrets
docker run --rm -v $(pwd):/src \
  secretscanner/secret-scanner:latest /src
```

**Dependency Scanning**:
```bash
# Scan PHP dependencies
make run CMD="composer audit"

# Check for outdated packages
make run CMD="composer outdated"
```

### Security Metrics

**Key Performance Indicators**:
- Failed authentication attempts per hour
- Unusual database query patterns
- Container restart frequency
- Network connection anomalies
- Secret access frequency

## Compliance Considerations

### Data Protection Regulations

**GDPR Compliance**:
- User consent management
- Right to data portability
- Right to be forgotten
- Data processing transparency

**Implementation**:
```bash
# Export user data (GDPR Article 20)
make run CMD="php artisan firefly-iii:export-user --user=user@example.com"

# Delete user data (GDPR Article 17)
make run CMD="php artisan firefly-iii:delete-user --user=user@example.com --confirm"
```

### Financial Data Protection

**Industry Standards**:
- PCI DSS compliance (if handling payment data)
- SOX compliance (if applicable)
- Financial data encryption requirements
- Audit trail maintenance

**Audit Logging**:
```bash
# Enable comprehensive audit logging
make run CMD="php artisan firefly-iii:enable-audit-log"

# Generate audit reports
make run CMD="php artisan firefly-iii:audit-report --from=2024-01-01 --to=2024-01-31"
```

### Security Documentation

**Required Documentation**:
- Security policies and procedures
- Incident response plans
- Access control matrices
- Data flow diagrams
- Risk assessments

## Security Checklist

### Deployment Security Checklist

- [ ] Strong, unique passwords for all accounts
- [ ] Multi-factor authentication enabled
- [ ] HTTPS/TLS configured with valid certificates
- [ ] Database access restricted to application only
- [ ] Container security hardening applied
- [ ] Security monitoring and alerting configured
- [ ] Regular backup and recovery testing
- [ ] Vulnerability scanning scheduled
- [ ] Security patch management process defined
- [ ] Incident response procedures documented

### Regular Security Maintenance

**Weekly Tasks**:
- [ ] Review security logs for anomalies
- [ ] Check for failed authentication attempts
- [ ] Verify backup integrity
- [ ] Update security signatures

**Monthly Tasks**:
- [ ] Run vulnerability scans
- [ ] Review and rotate secrets
- [ ] Update security documentation
- [ ] Test incident response procedures

**Quarterly Tasks**:
- [ ] Security assessment and penetration testing
- [ ] Review and update security policies
- [ ] Audit user access and permissions
- [ ] Update threat model and risk assessment

---

**Sources**:
- Firefly III Security Documentation - https://docs.firefly-iii.org/how-to/firefly-iii/security/ (Retrieved: 2024-01-15)
- OWASP Application Security Guidelines - https://owasp.org/www-project-application-security-verification-standard/ (Retrieved: 2024-01-15)
- Docker Security Best Practices - https://docs.docker.com/engine/security/ (Retrieved: 2024-01-15)
- Laravel Security Documentation - https://laravel.com/docs/11.x/security (Retrieved: 2024-01-15)