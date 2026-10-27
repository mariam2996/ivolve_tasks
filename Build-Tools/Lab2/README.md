# Lab 2 - Building and Packaging Java Application with Gradle

## Objective
Build, test, package, and run a Java calculator application using Gradle.

## Source Code
https://github.com/Ibrahim-Adel15/CalculatorGradleApp.git

## Run Unit Tests

Gradle and Java were executed using a Gradle Docker image.

```bash
docker run --rm \
  -v "$(pwd)":/home/gradle/project \
  -w /home/gradle/project \
  gradle:8.10-jdk17 \
  gradle test
```

## Build the Application

```bash
docker run --rm \
  -v "$(pwd)":/home/gradle/project \
  -w /home/gradle/project \
  gradle:8.10-jdk17 \
  gradle clean build
```

The generated artifact is:

```text
build/libs/calculator.jar
```

## Run the Application

```bash
docker run --rm -it \
  -v "$(pwd)":/home/gradle/project \
  -w /home/gradle/project \
  gradle:8.10-jdk17 \
  java -jar build/libs/calculator.jar
```

## Verification

Example input:

```text
First number: 5
Second number: 10
```

Example output:

```text
Sum: 15.0
Subtract: -5.0
Multiply: 50.0
Divide: 0.5
```

## Result

The Java application was successfully tested, built, packaged into a JAR file, and executed using Gradle.
