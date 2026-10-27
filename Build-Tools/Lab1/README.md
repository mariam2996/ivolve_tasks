# Lab 1 - Building and Packaging Java Application with Maven

## Objective
Build, test, package, and run a Java calculator application using Maven.

## Source Code
https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git

## Run Unit Tests

Maven and Java were executed using a Maven Docker image.

```bash
docker run --rm \
  -v "$(pwd)":/app \
  -w /app \
  maven:3.9.9-eclipse-temurin-17 \
  mvn test
