# Multi-stage build for production
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./

# Changed this line ←←←←←←←←←←←←←←←←
RUN npm install --production && npm cache clean --force
# Instead of: RUN npm ci --only=production ...

COPY . .

RUN rm -rf node_modules/.cache

# Production stage
FROM node:18-alpine AS production

RUN apk add --no-cache curl

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./
COPY --chown=nodejs:nodejs . .

RUN mkdir -p /app/data /app/cache /app/logs && chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:3001/api/health || exit 1

CMD ["node", "index.js"]
