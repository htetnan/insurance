FROM maven:3.9.11-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -B -DskipTests dependency:go-offline
COPY src ./src
RUN mvn -B -DskipTests clean package

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN mkdir -p /data/uploads
ENV FILE_STORAGE_DIR=/data/uploads
EXPOSE 8081
CMD ["sh", "-c", "java -jar app.jar"]
