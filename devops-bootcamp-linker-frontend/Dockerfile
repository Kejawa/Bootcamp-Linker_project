# Install dependencies && Build
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

# Copy rest of source code
COPY . .

# Define a build argument for API URL used by frontend
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL

# Build the Next.js application for production
RUN npm run build

# Production runtime
FROM node:20-alpine AS runner

WORKDIR /app 

ENV NODE_ENV=production

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

COPY --from=builder --chown=appuser:appgroup /app/.next/standalone ./
COPY --from=builder --chown=appuser:appgroup /app/.next/static ./.next/static 
COPY --from=builder --chown=appuser:appgroup /app/public ./public 

USER appuser

EXPOSE 3000

CMD ["node", "server.js"]
