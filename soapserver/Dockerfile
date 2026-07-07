# Modernized 2026-07: AdoptOpenJDK (archived) -> Eclipse Temurin; openjdk:8-jdk-alpine
# -> temurin 8 JRE; fully-qualified image names (buildah enforces short-name resolution).
FROM docker.io/library/eclipse-temurin:8-jdk AS builder

COPY . /code

WORKDIR /code

RUN ./gradlew clean build


FROM docker.io/library/eclipse-temurin:8-jre

COPY --from=builder /code/build/libs/*.jar app.jar

ENTRYPOINT ["java","-jar","/app.jar"]
