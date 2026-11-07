# Travelo database image for Docker Swarm:
# the official MySQL image + the seed script baked in, so every replica starts with the same data
# (Swarm services cannot easily bind-mount a host file, and Swarm configs are limited to 500 KB).
FROM mysql:8.0
LABEL org.opencontainers.image.title="travelo-database" \
      org.opencontainers.image.authors="Oualid AMTOUT"
COPY init.sql /docker-entrypoint-initdb.d/init.sql
