# ---------- Builder ----------
FROM node:24-alpine AS builder

WORKDIR /app

# Copy package manifests first for better caching
COPY common/package*.json ./common/
COPY next-client/package*.json ./next-client/

# Install deps
WORKDIR /app/common
RUN npm ci

WORKDIR /app/next-client
RUN npm ci

# Copy source
WORKDIR /app
COPY common ./common
COPY next-client ./next-client

# Build shared package first
WORKDIR /app/common
RUN npm run build

# Build Next.js app
WORKDIR /app/next-client
RUN npm run build


# ---------- Runner ----------
FROM node:24-alpine AS runner

ENV NODE_ENV=production

WORKDIR /app

# Next.js standalone server
COPY --from=builder /app/next-client/.next/standalone ./

# Static assets
COPY --from=builder /app/next-client/.next/static ./.next/static
COPY --from=builder /app/next-client/public ./public

EXPOSE 3000

CMD ["node", "server.js"]
