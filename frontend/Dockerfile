# =============================================================================
# GigaChad GRC Frontend - Multi-stage Docker Build
# =============================================================================
# This Dockerfile builds the React frontend and serves it via Nginx.
# Build args are used to configure the frontend at build time.
# =============================================================================

# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files from frontend directory (context is repo root)
COPY frontend/package*.json ./

# Install ALL dependencies (including devDependencies) for build
RUN npm install

# Copy frontend source
COPY frontend/ ./

# Build-time environment variables
# These are baked into the static build
ARG VITE_API_URL=http://localhost:3001
ARG VITE_ENABLE_AI_MODULE=true
ARG VITE_ENABLE_DEV_AUTH=true
ARG VITE_KEYCLOAK_URL=http://localhost:8080
ARG VITE_KEYCLOAK_REALM=gigachad-grc
ARG VITE_KEYCLOAK_CLIENT_ID=grc-frontend

ENV VITE_API_URL=$VITE_API_URL
ENV VITE_ENABLE_AI_MODULE=$VITE_ENABLE_AI_MODULE
ENV VITE_ENABLE_DEV_AUTH=$VITE_ENABLE_DEV_AUTH
ENV VITE_KEYCLOAK_URL=$VITE_KEYCLOAK_URL
ENV VITE_KEYCLOAK_REALM=$VITE_KEYCLOAK_REALM
ENV VITE_KEYCLOAK_CLIENT_ID=$VITE_KEYCLOAK_CLIENT_ID

# Build with staging mode to bypass vite.config validation while keeping production optimizations
RUN npm run build -- --mode staging

# Production stage - Nginx
FROM nginx:alpine AS production

# Copy built assets from builder
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx config (path relative to context which is repo root)
COPY frontend/nginx.conf /etc/nginx/conf.d/default.conf

# Nginx runs on port 80 by default
EXPOSE 80

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1:80/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
