# syntax=docker/dockerfile:1

# ======================= Stage 1: build =======================
# Node.js is needed only to compile the React application into static files
FROM node:20-alpine AS build
WORKDIR /app

# 1) Dependencies first (cached layer as long as package*.json do not change)
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm npm ci --no-audit --no-fund

# 2) Source code, then production build (optimised, minified bundle in /app/build)
COPY public ./public
COPY src ./src
RUN npm run build

# ======================= Stage 2: runtime =======================
# Unprivileged Nginx: runs as the non-root user "nginx" (uid 101), Alpine-based
FROM nginxinc/nginx-unprivileged:stable-alpine

LABEL org.opencontainers.image.title="travelo-frontend" \
      org.opencontainers.image.description="Travelo flight booking - React frontend served by Nginx" \
      org.opencontainers.image.version="1.0" \
      org.opencontainers.image.authors="Oualid AMTOUT"

# Default backend address (overridable at run time with -e BACKEND_URL=...)
ENV BACKEND_URL=http://travelo-backend:8080

# Nginx configuration template (rendered into /etc/nginx/conf.d/default.conf at start)
COPY --chown=nginx:nginx nginx.conf.template /etc/nginx/templates/default.conf.template

# Only the compiled static files: no Node.js, no node_modules, no source code
COPY --from=build --chown=nginx:nginx /app/build /usr/share/nginx/html

USER nginx

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3000/ > /dev/null || exit 1

# The base image's ENTRYPOINT renders the templates, then CMD starts Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]
