# syntax=docker/dockerfile:1

FROM node:20-alpine AS frontend-build
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ ./
RUN npm run build

FROM node:20-alpine AS backend-build
WORKDIR /app/backend
COPY backend/package*.json ./
RUN npm install --omit=dev
COPY backend/ ./

FROM node:20-alpine AS runtime
RUN addgroup -S appgroup && adduser -S appuser -G appgroup && apk add --no-cache wget
WORKDIR /app

COPY --from=backend-build /app/backend ./backend
COPY --from=frontend-build /app/frontend/build ./backend/public

ENV NODE_ENV=production
ENV PORT=5000

USER appuser
EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD wget -qO- http://localhost:5000/health || exit 1

CMD ["node", "backend/server.js"]
