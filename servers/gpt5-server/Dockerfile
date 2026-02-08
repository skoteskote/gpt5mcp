# ---- build stage ----
FROM node:20-alpine AS build
WORKDIR /app

# 1) Install ALL deps (including dev) so tsc is available
COPY package*.json ./
RUN npm ci

# 2) Build
COPY tsconfig.json ./
COPY src ./src
RUN npx tsc

# 3) Prune dev deps so node_modules is prod-only for runtime
RUN npm prune --omit=dev

# ---- runtime stage ----
FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production

# Non-root
RUN addgroup -S app && adduser -S app -G app
USER app

# Copy only what we need to run
COPY --from=build /app/package*.json ./
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/build ./build

# Env toggles (do NOT bake secrets)
ENV OPENAI_API_KEY=""
ENV DEBUG_MCP="0"

CMD ["node", "build/index.js"]
