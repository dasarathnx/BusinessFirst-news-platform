# Base image
FROM node:20-alpine AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

# Step 1: Install dependencies
FROM base AS dependencies
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml tsconfig.base.json ./
COPY apps/api/package.json ./apps/api/
COPY packages ./packages
RUN pnpm install --frozen-lockfile

# Step 2: Build the application
FROM dependencies AS builder
WORKDIR /app
COPY apps/api ./apps/api
RUN pnpm --filter @businessfirst/api db:generate
RUN pnpm --filter @businessfirst/api build

# Step 3: Production runner
FROM base AS runner
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml tsconfig.base.json ./
COPY packages ./packages
COPY apps/api/package.json ./apps/api/
RUN pnpm install --prod --frozen-lockfile

# Copy build outputs from builder
COPY --from=builder /app/apps/api/dist ./apps/api/dist
COPY --from=builder /app/apps/api/src/generated/prisma ./apps/api/src/generated/prisma

WORKDIR /app/apps/api
EXPOSE 8083
ENV PORT=8083
ENV NODE_ENV=production

CMD ["node", "dist/server.js"]
