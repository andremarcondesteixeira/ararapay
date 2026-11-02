FROM amazoncorretto:27-jdk AS build
RUN dnf install -y tar gzip && dnf clean all
WORKDIR /app
COPY .mvn .mvn
COPY mvnw pom.xml ./

# Download all dependencies before copying the source code.
# This layer is cached until pom.xml changes, so editing code doesn't re-download everything.
# -B = batch mode (non-interactive, no colored/progress output in logs).
RUN ./mvnw dependency:go-offline -B

COPY src src
RUN ./mvnw clean package -DskipTests -B

FROM amazoncorretto:27-alpine
WORKDIR /app
COPY --from=build /app/target/api.jar api.jar
CMD ["java", "-jar", "api.jar"]
