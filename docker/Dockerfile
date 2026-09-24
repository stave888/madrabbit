# Multi-stage build for MadRabbit vulnerability lab platform
FROM maven:3.8-eclipse-temurin-17 AS builder

# Set working directory
WORKDIR /app

# Copy pom.xml and download dependencies (cache layer)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre

# Set working directory
WORKDIR /app

# Copy the built JAR from builder stage
COPY --from=builder /app/target/madrabbit-1.0.0.jar app.jar

# Copy additional resources
COPY uploads ./uploads
COPY downloads ./downloads
COPY secret ./secret

# Create necessary directories
RUN mkdir -p /app/logs

# Expose application port
EXPOSE 8080

# JVM options for deserialization challenges
ENV JAVA_OPTS="--add-opens java.naming/javax.naming=ALL-UNNAMED \
--add-opens java.base/java.lang=ALL-UNNAMED \
--add-opens java.base/java.lang.reflect=ALL-UNNAMED"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/login.html || exit 1

# Run the application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
