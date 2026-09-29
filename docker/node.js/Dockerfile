# Stage 1: Build TypeScript
FROM node:22-alpine AS builder

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@latest --activate

RUN --mount=type=cache,target=/root/.pnpm-store \
    --mount=type=bind,source=package.json,target=/app/package.json \
    --mount=type=bind,source=pnpm-lock.yaml,target=/app/pnpm-lock.yaml \
    --mount=type=bind,source=pnpm-workspace.yaml,target=/app/pnpm-workspace.yaml \
    pnpm install --frozen-lockfile

COPY . .
RUN pnpm run build


# Stage 2: Install Production Dependencies
FROM node:22-alpine AS deps

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@latest --activate

RUN --mount=type=cache,target=/root/.pnpm-store \
    --mount=type=bind,source=package.json,target=/app/package.json \
    --mount=type=bind,source=pnpm-lock.yaml,target=/app/pnpm-lock.yaml \
    --mount=type=bind,source=pnpm-workspace.yaml,target=/app/pnpm-workspace.yaml \
    pnpm install --frozen-lockfile --prod


# Stage 3: Final Runner Image
FROM node:22-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

COPY --from=deps --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/dist ./dist
COPY --from=builder --chown=node:node /app/package.json ./package.json

USER node

EXPOSE 3000

CMD ["node", "dist/index.js"]