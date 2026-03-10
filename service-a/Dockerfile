# syntax=docker/dockerfile:1.7

########################
# Stage 1: Build (Maven)
########################
FROM maven:3.9.9-eclipse-temurin-17 AS build
WORKDIR /app

# Copy pom dan prefetch dependencies untuk cache efisien
COPY pom.xml .
RUN --mount=type=cache,target=/root/.m2 \
    mvn -B -q -DskipTests dependency:go-offline

# Copy source dan build jar
COPY src ./src
RUN --mount=type=cache,target=/root/.m2 \
    mvn -B -DskipTests package

########################
# Stage 2: Runtime (JRE)
########################
FROM eclipse-temurin:17-jre-jammy AS runtime
ENV APP_HOME=/opt/app
WORKDIR ${APP_HOME}

# Buat user non-root untuk security
RUN useradd --no-create-home --uid 10001 appuser

# Salin hasil build dari stage build
# Sesuaikan nama file JAR jika berbeda
COPY --from=build /app/target/service-a-0.0.1-SNAPSHOT.jar app.jar

# Port default Spring Boot
EXPOSE 8080

# Opsi JVM yang ramah container + profile default
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75 -XX:InitialRAMPercentage=50 -Djava.security.egd=file:/dev/./urandom" \
    SPRING_PROFILES_ACTIVE=prod

USER 10001

ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar app.jar"]