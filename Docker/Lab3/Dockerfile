FROM maven:3.9.9-eclipse-temurin-17

WORKDIR /app

RUN groupadd --system appgroup && \
    useradd --system --gid appgroup --create-home appuser

COPY . .

RUN chown -R appuser:appgroup /app

USER appuser

RUN mvn clean package

CMD ["java", "-jar", "target/calculator.jar"]
