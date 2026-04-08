# Stage 1: Build the application
FROM node:20-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json* svelte.config.js vite.config.ts* tsconfig.json* ./
RUN npm ci --ignore-scripts

COPY . .
RUN npm run build
RUN npm prune --production

# Stage 2: Create the production image
FROM node:20-alpine
WORKDIR /app

COPY --from=builder /app/build ./build
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/package-lock.json ./package-lock.json

# Copy drizzle config and schema for drizzle-kit push
COPY --from=builder /app/drizzle.config.ts ./drizzle.config.ts
COPY --from=builder /app/src/lib/server/db/schema.ts ./src/lib/server/db/schema.ts
COPY --from=builder /app/tsconfig.json ./tsconfig.json

# Install drizzle-kit (and its deps like tsx) after node_modules are in place
RUN npm install --no-save drizzle-kit tsx

EXPOSE 3000
ENV NODE_ENV=production

CMD ["sh", "-c", "npx drizzle-kit push && node build"]
