# Shared Dockerfile for every Java (Gradle) service. Every service listens on 3000.
# Service repo needs: settings.gradle, build.gradle, gradle.properties, src/, libs/, fonts/
# (libs/ and fonts/ may be empty, keep a .gitkeep). The build must produce one *-SNAPSHOT.jar.
FROM gradle:9.5.1-jdk21-alpine AS builder

WORKDIR /app

COPY settings.gradle build.gradle gradle.properties ./

COPY libs ./libs
COPY fonts ./fonts

RUN gradle dependencies --no-daemon || true
COPY src ./src

RUN gradle clean build -x test -P skipCheckstyle=true --no-daemon

########################
FROM eclipse-temurin:21.0.13_9-jre-alpine-3.24

ENV TZ="UTC"
ARG CI_TAG="dev"
ENV APP_VERSION=$CI_TAG
# Spring Boot reads SERVER_PORT: same port as the chart (service.http.port) and the probes
ENV SERVER_PORT=3000

# The JRE image has no "gitops" user: create it, otherwise the container fails to start
# app dir owned by gitops: -XX:+HeapDumpOnOutOfMemoryError writes the dump into the working dir
RUN addgroup -S gitops && adduser -S -G gitops -h /home/gitops gitops \
    && mkdir -p /home/gitops/app && chown gitops:gitops /home/gitops/app

WORKDIR /home/gitops/app

COPY --from=builder /app/fonts /usr/share/fonts/

COPY --from=builder --chown=gitops:gitops /app/build/libs/*-SNAPSHOT.jar application.jar

USER gitops
EXPOSE 3000

CMD java \
    -Djava.awt.headless=true \
    -Duser.timezone=UTC \
    -Djava.security.egd=file:/dev/./urandom  \
    -Xms128m \
    -XX:MaxMetaspaceSize=256m \
    -server \
    -XX:+HeapDumpOnOutOfMemoryError \
    -jar /home/gitops/app/application.jar