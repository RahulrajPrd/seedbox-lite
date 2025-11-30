# Simple, reliable production build – no package-lock.json needed
FROM node:18-alpine AS base

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies (works without package-lock.json)
RUN npm install --production --silent && npm cache clean --force

# Copy source code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Install curl for healthcheck
RUN apk add --no-cache curl

# Create required directories
RUN mkdir -p /app/data /app/cache /app/logs

# Fix ownership
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:3001/api/health || exit 1

CMD ["node", "index.js"]
