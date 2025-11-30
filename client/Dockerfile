# Multi-stage build for React/Vite frontend
FROM node:18-alpine AS builder

# We are building from repo root context, so client/ files are in ./client/
WORKDIR /app

# Copy ONLY the client folder into the image
COPY client ./ 

# Now switch into the client directory – this makes everything work
WORKDIR /app/client

# Install dependencies
COPY client/package*.json ./
RUN npm ci && npm cache clean --force

# Copy the rest of the client code
COPY client/. .

# Build-time env var
ARG VITE_API_BASE_URL=http://localhost:3001
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

# Build the app
RUN npm run build

# ──────────────────────────────
# Production stage – Nginx
# ──────────────────────────────
FROM nginx:alpine AS production

RUN apk add --no-cache curl

# This line now works because nginx.conf is inside /app/client/
COPY --from=builder /app/client/nginx.conf /etc/nginx/nginx.conf

# Copy the built static files
COPY --from=builder /app/client/dist /usr/share/nginx/html

# Nginx non-root setup
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
