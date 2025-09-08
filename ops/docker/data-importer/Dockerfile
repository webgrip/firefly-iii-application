# Use official Firefly III Data Importer image with exact version tag
FROM fireflyiii/data-importer:version-1.5.4

# Minimal overlay for project-specific configuration
# The base image already contains all necessary components for data importing
# Including PHP-FPM, Laravel application, and required extensions

# Health check to ensure the data importer is running
HEALTHCHECK --interval=30s --timeout=3s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/ || exit 1

# The base image already exposes port 8080 and runs as www-data user
# No additional configuration needed as upstream image handles everything