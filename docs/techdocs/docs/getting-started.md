# Getting Started

**Purpose**: This guide provides step-by-step instructions for setting up and running the Firefly III application stack for the first time.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [First Run](#first-run)
- [Initial Configuration](#initial-configuration)
- [Creating Your First User](#creating-your-first-user)
- [Next Steps](#next-steps)

## Prerequisites

Before starting, ensure you have the following installed:

- **Docker**: Version 20.10 or later
- **Docker Compose**: Version 2.0 or later (or Docker with compose plugin)
- **Make**: GNU Make utility
- **Git**: For cloning the repository

### System Requirements

- **Memory**: Minimum 1GB RAM, recommended 2GB+
- **Storage**: At least 5GB free disk space
- **CPU**: Any modern x64 or ARM64 processor

## Initial Setup

### 1. Clone the Repository

```bash
git clone https://github.com/webgrip/firefly-iii-application.git
cd firefly-iii-application
```

### 2. Environment Configuration

Copy the example environment file and customize it:

```bash
cp .env.example .env
```

Edit the `.env` file to configure basic settings:

```bash
# Application URL (change to your domain in production)
BASE_URL=http://localhost:8080
APP_URL=http://localhost:8080

# Application settings
APP_ENV=local
APP_DEBUG=true
APP_LOCALE=en
APP_TIMEZONE=Europe/Amsterdam

# Generate application key (see below)
APP_KEY=

# Database configuration (defaults work for local development)
DB_CONNECTION=mysql
DB_HOST=firefly-iii-application.mariadb
DB_PORT=3306
DB_DATABASE=application
DB_USERNAME=application
DB_PASSWORD=application
```

### 3. Generate Application Key

Generate a secure application key for Laravel:

```bash
# This will be automated in future versions
# For now, generate a 32-character random string
echo "APP_KEY=base64:$(openssl rand -base64 32)" >> .env
```

## First Run

### 1. Start the Services

```bash
make start
```

This command will:
- Build the Docker images
- Start MariaDB, Redis, Application, and Nginx containers
- Initialize the database schema
- Configure the application

### 2. Wait for Services to Initialize

Monitor the startup process:

```bash
make logs
```

Wait for these indicators that services are ready:
- MariaDB: "ready for connections"
- Redis: "Ready to accept connections"
- Application: "NOTICE: ready to handle connections"
- Nginx: "nginx: [notice] start worker processes"

### 3. Verify Application Access

Open your browser and navigate to:
```
http://localhost:8080
```

You should see the Firefly III setup wizard.

## Initial Configuration

### 1. Database Setup

The database will be automatically initialized with:
- Firefly III database schema
- Default settings and configurations
- Required system data

### 2. Application Configuration

Follow the Firefly III setup wizard to configure:
- Site name and description
- Default currency
- Initial settings
- Administrative user account

## Creating Your First User

### Option 1: Through Web Interface

1. Complete the setup wizard
2. Create an admin account when prompted
3. Log in with your new credentials

### Option 2: Command Line (Alternative)

```bash
# Create user via command line
make user:create EMAIL=admin@example.com PASS=yourpassword
```

## Next Steps

After successful setup:

1. **Read the Configuration Guide**: Learn about environment variables and advanced settings
2. **Review Security Settings**: Configure proper security for production use
3. **Set Up Backups**: Implement data backup procedures
4. **Explore Features**: Start using Firefly III's financial management features

### Useful Commands

```bash
# View logs from all services
make logs

# View logs from specific service
make logs SERVICE=firefly-iii-application.application

# Stop all services
make stop

# Enter application container for debugging
make enter

# Run one-off commands in application container
make run CMD="php artisan --version"
```

## Troubleshooting

### Common Issues

**Services not starting**: Check Docker is running and ports 8080 and 3306 are available.

**Database connection errors**: Ensure MariaDB container is healthy before application starts.

**Permission errors**: Verify Docker has proper permissions to bind mount volumes.

For detailed troubleshooting, see the [Troubleshooting Guide](troubleshooting.md).

---

**Sources**:
- Firefly III Official Documentation - https://docs.firefly-iii.org/ (Retrieved: 2024-01-15)
- Docker Compose Documentation - https://docs.docker.com/compose/ (Retrieved: 2024-01-15)