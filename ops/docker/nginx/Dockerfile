# Use official nginx alpine image with exact pinned version
FROM nginx:1.26.2-alpine3.20

# Copy nginx configuration for Firefly III reverse proxy
COPY ops/docker/nginx/config/default.conf /etc/nginx/conf.d/default.conf

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/health || exit 1

# Use nginx default user and configuration
EXPOSE 80
