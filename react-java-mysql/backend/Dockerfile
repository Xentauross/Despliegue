# syntax=docker/dockerfile:1.4

#Build the application using Maven
FROM --platform=$BUILDPLATFORM maven:3.8.5-eclipse-temurin-17 AS builder
WORKDIR /workdir/server
COPY pom.xml /workdir/server/pom.xml
# Download dependencies to speed up subsequent builds 
RUN mvn dependency:go-offline

COPY src /workdir/server/src
# Compile and package the application
RUN mvn install

#  Development environment setup
FROM builder as dev-envs

# Install necessary development tools like git
RUN <<EOF
apt-get update
apt-get install -y git
EOF

# Create a non-root user for development
RUN <<EOF
useradd -s /bin/bash -m vscode
groupadd docker
usermod -aG docker vscode
EOF

# Install Docker tools (cli, buildx, compose) from a specialized image
COPY --from=gloursdocker/docker / /
CMD ["mvn", "spring-boot:run"]

# Prepare files for production by extracting the JAR
FROM builder as prepare-production
RUN mkdir -p target/dependency
WORKDIR /workdir/server/target/dependency
# Extract the JAR to separate libraries from application code
RUN jar -xf ../*.jar

#Final production image (minimal JRE)
FROM eclipse-temurin:17-jre-focal

EXPOSE 8080
VOLUME /tmp
ARG DEPENDENCY=/workdir/server/target/dependency

# Copy extracted dependencies and classes from the preparation stage
COPY --from=prepare-production ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=prepare-production ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=prepare-production ${DEPENDENCY}/BOOT-INF/classes /app

# Run the application using the specific classpath to optimize startup time
ENTRYPOINT ["java","-cp","app:app/lib/*","com.company.project.Application"]