# Build stage with Maven and JDK 11
FROM maven:3.8.5-eclipse-temurin-11 AS builder

WORKDIR /app

# Cache dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Build app
COPY src ./src
RUN mvn install -DskipTests

# Prepare unpacked JAR
RUN mkdir -p target/dependency
RUN jar -xf ./target/*.jar


# Slim production image with JRE only
FROM eclipse-temurin:11-jre-focal

VOLUME /tmp

# Copy unpacked app contents
COPY --from=builder /app/META-INF /app/META-INF
COPY --from=builder /app/BOOT-INF/classes /app
COPY --from=builder /app/BOOT-INF/lib /app/lib

EXPOSE 80

# Run the application
ENTRYPOINT [ "java", "-cp","app:app/lib/*", "com.chattingo.ChattingoApplication" ]
