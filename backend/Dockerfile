# ===== Build stage =====
FROM maven:3.9-eclipse-temurin-11 AS builder
WORKDIR /app
COPY pom.xml .
# Pre-cache dependencies
RUN mvn -B -q -e -DskipTests dependency:go-offline
COPY src ./src
# Build
RUN mvn -B -q -DskipTests clean package

# ===== Runtime stage =====
FROM eclipse-temurin:11-jre
WORKDIR /app
# Copy built JAR
ARG JAR_FILE=target/*.jar
COPY --from=builder /app/${JAR_FILE} /app/app.jar

# Spring Boot defaults to 8080
EXPOSE 8080
# Use environment overrides for DB connection
ENV JAVA_OPTS=""
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/app.jar"]
