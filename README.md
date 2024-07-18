# Building and Pushing Docker Images for a Spring Boot Application with Jib

This guide demonstrates how to use Jib to build and push Docker images for a Spring Boot application. The application is built using Gradle, targeting Java 17, and utilizes Spring Boot version 3.3.1.

## Prerequisites

- Docker account (for pushing images)
- GitHub repository (to host your project and workflows)
- Java 17
- Gradle

## Project Setup

Ensure your project has the following structure and configuration.

### `build.gradle`

Include the necessary plugins and dependencies in your `build.gradle` file:

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.1'
    id 'io.spring.dependency-management' version '1.1.5'
    id 'com.google.cloud.tools.jib' version '3.4.0'
}

group = 'clovider'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}

jib {
    from {
        image = 'eclipse-temurin:17-jre'
    }
    to {
        image = 'mango0422/testjib'
        tags = ['latest', version]
    }
    container {
        ports = ['8080']
        jvmFlags = ['-Xms512m', '-Xmx512m']
    }
}
```

### GitHub Actions Workflow

Create a GitHub Actions workflow file at `.github/workflows/gradle.yml` to automate the build and push process:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Validate Gradle wrapper
        uses: gradle/wrapper-validation-action@v1

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build and push image with Jib
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
        run: |
          ./gradlew jib \
            -Djib.to.auth.username=$DOCKER_USERNAME \
            -Djib.to.auth.password=$DOCKER_PASSWORD
```

### Secrets Configuration

Ensure you have set the necessary secrets in your GitHub repository settings:

- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub password.

To set these secrets, navigate to your repository on GitHub, go to **Settings > Secrets and variables > Actions** and add `DOCKER_USERNAME` and `DOCKER_PASSWORD`.

## Running the Workflow

Once the above configuration is in place, every push to the `main` branch will trigger the GitHub Actions workflow. This workflow will:

1. Check out the repository code.
2. Validate the Gradle wrapper.
3. Set up JDK 17.
4. Grant execute permissions to `gradlew`.
5. Build and push the Docker image using Jib.

## Conclusion

With this setup, you can seamlessly build and push Docker images for your Spring Boot application using Jib, Gradle, and GitHub Actions. This automation ensures that your Docker images are consistently built and pushed to your Docker Hub repository with every change pushed to the `main` branch.