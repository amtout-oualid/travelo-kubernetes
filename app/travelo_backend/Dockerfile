# syntax=docker/dockerfile:1

# ======================= Stage 1: build =======================
# Full Maven + JDK image, used ONLY to compile. It is not part of the final image.
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /build

# 1) Copy only the dependency descriptor first and download dependencies.
#    This layer is cached and reused as long as pom.xml does not change.
COPY pom.xml .
RUN --mount=type=cache,target=/root/.m2 mvn -B -q dependency:go-offline

# 2) Copy the source code and package the application (tests are run in CI, not here)
COPY src ./src
RUN --mount=type=cache,target=/root/.m2 mvn -B -q package -DskipTests \
    && cp target/*.jar app.jar

# 3) Split the fat jar into layers (dependencies change rarely, application code often)
RUN java -Djarmode=tools -jar app.jar extract --layers --launcher --destination extracted

# ======================= Stage 2: runtime =======================
# Minimal image: Java Runtime only (no JDK, no Maven), on Alpine Linux, pinned major version
FROM eclipse-temurin:21-jre-alpine

LABEL org.opencontainers.image.title="travelo-backend" \
      org.opencontainers.image.description="Travelo flight booking - Spring Boot REST API" \
      org.opencontainers.image.version="1.0" \
      org.opencontainers.image.authors="Oualid AMTOUT"

# Apply the latest security patches of the base OS packages
RUN apk upgrade --no-cache

# Dedicated unprivileged user: the application never runs as root
RUN addgroup -S travelo && adduser -S travelo -G travelo

WORKDIR /app

# Copy the layers from the least to the most frequently changing one (better cache reuse)
COPY --from=build --chown=travelo:travelo /build/extracted/dependencies/ ./
COPY --from=build --chown=travelo:travelo /build/extracted/spring-boot-loader/ ./
COPY --from=build --chown=travelo:travelo /build/extracted/snapshot-dependencies/ ./
COPY --from=build --chown=travelo:travelo /build/extracted/application/ ./

USER travelo

# JVM tuned for containers: heap sized from the container memory limit
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75.0 -XX:+ExitOnOutOfMemoryError"

# Documentation of the port the API listens on
EXPOSE 8080

# Docker checks that the API really answers (busybox wget is included in Alpine)
HEALTHCHECK --interval=30s --timeout=5s --start-period=60s --retries=3 \
  CMD wget -qO- http://localhost:8080/api/hello || exit 1

# Exec form: Java is PID 1 and receives the stop signals (graceful shutdown)
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
