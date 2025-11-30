# Multi-stage build for React/Vite frontend
FROM node:18-alpine AS builder

# FIX: Set context to the client folder itself
WORKDIR /app
COPY . ./client/
WORKDIR /app/client

# Now everything is in the right place
COPY package*.json ./
RUN npm ci && npm cache clean --force

COPY . .

ARG VITE_API_BASE_URL=http://localhost:3001
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

RUN npm run build

# Production stage
FROM nginx:alpine AS production

RUN apk add --no-cache curl

# This now finds nginx.conf because we're in the client folder
COPY nginx.conf /etc/nginx/nginx.conf

COPY --from=builder /app/client/dist /usr/share/nginx/html

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
