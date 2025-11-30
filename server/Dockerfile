# ---------- Builder Stage ----------
FROM node:18-alpine AS builder

WORKDIR /app

# Copy only package files first (best layer caching)
COPY package*.json ./

# Install ALL dependencies (including dev) for building
RUN npm ci --silent

# Copy source code
COPY . .

# Optional: Run build if you have one (e.g. TypeScript)
# RUN npm run build

# ---------- Prune Stage (remove dev dependencies) ----------
FROM node:18-alpine AS pruner

WORKDIR /app

COPY package*.json ./
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Install only production dependencies (removes devDependencies)
RUN npm ci --only=production --silent && npm cache clean --force

# ---------- Final Production Image ----------
FROM node:18-alpine AS production

# Install curl for healthcheck
RUN apk add --no-cache curl

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

# Copy only production node_modules and app code
COPY --from=pruner --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=pruner --chown=nodejs:nodejs /app/package*.json ./
COPY --chown=nodejs:nodejs . .

# Create required directories
RUN mkdir -p /app/data /app/cache /app/logs && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:3001/api/health || exit 1

CMD ["node", "index.js"]
