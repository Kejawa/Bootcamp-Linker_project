# Stage 1: Build
# Node 20 Alpine image for building the app
FROM node:20-alpine AS builder

WORKDIR /app

# Install build packages required
RUN apk add --no-cache python3 build-base openssl

# Copy and Install all dependencies
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .

# Generate prisma client. Build the app
RUN npx prisma generate
RUN npm run build


# Stage 2: Production
FROM node:20-alpine AS runner

# set environment
ENV NODE_ENV=production \
    PORT=3001

WORKDIR /app 

# Install runtime deps
RUN apk add --no-cache openssl curl

# Run as non root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy package files
COPY --from=builder --chown=appuser:appgroup /app/package*.json ./

# Install ONLY production dependencies here
# instead of copying all node_modules from builder
RUN npm ci --omit=dev && npm cache clean --force

# Copy generated prisma Client
COPY --from=builder --chown=appuser:appgroup /app/node_modules/@prisma ./node_modules/@prisma
COPY --from=builder --chown=appuser:appgroup /app/node_modules/.prisma ./node_modules/.prisma


# Copy built application & prisma client
COPY --from=builder --chown=appuser:appgroup /app/prisma ./prisma
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/entrypoint-script.sh ./entrypoint-script.sh


RUN chmod +x ./entrypoint-script.sh

USER appuser

EXPOSE 3001

CMD ["./entrypoint-script.sh"]