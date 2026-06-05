---
name: create-java-gradle-devcontainer-project
description: Create a Java Gradle Kotlin DSL project from a Docker Dev Container contract. Use when Codex should scaffold a new Java project, declare the containerized Java and Gradle environment, generate Gradle Wrapper through Docker, add `.devcontainer/devcontainer.json`, create application code and tests, and validate the project through Dev Container CLI.
---

# Create Java Gradle Devcontainer Project

## Overview

This skill creates a runnable Java Gradle Kotlin DSL project by declaring the Docker Dev Container environment first.

The container environment is the source of truth:

- Java image: `mcr.microsoft.com/devcontainers/java:25`
- Gradle wrapper image: `gradle:9.2.1-jdk25`
- Gradle Wrapper version: `9.2.1`
- Build files: Kotlin DSL, `build.gradle.kts`
- Validation: `devcontainer up`

This skill exists to generate a runnable project. The specific business domain of the project is not important. Start from the Docker environment contract, then create the project around it.

## When To Use

Use this skill when the user wants a new Java project that is developed, built, tested, and run inside Docker Dev Container.

Good examples:

- Create a Java Gradle project that runs in Docker Dev Container.
- Create a Spring Boot project with Java 25 and Dev Container support.
- Create a Java project with an HTTP endpoint and tests, validated in Dev Container.
- Scaffold a Docker-first Java Gradle sample project.

## Hard Constraints

These rules are mandatory for this skill:

1. Docker Dev Container is the only supported execution model.
   - Always create, build, test, and validate the project through Docker Dev Container.
   - Do not switch to host Java, host Gradle, or any non-container fallback.
   - If Docker or `devcontainer` is unavailable, treat that as an environment limitation rather than changing the workflow.
2. For Spring Boot projects, tests must use the native Spring Boot style.
   - Use `@SpringBootTest`.
   - Do not replace Spring Boot integration testing with standalone controller tests just to work around configuration issues.
   - If a Spring Boot test fails because of dependencies or configuration, fix the project template so `@SpringBootTest` works.

## Inputs To Confirm

Confirm only the project facts needed to generate files:

1. Target parent directory.
2. Project name.
3. Java package name, if the user has one.
4. Project type:
   - plain Java application
   - Spring Boot web application
   - other user-provided architecture
5. Optional HTTP endpoint and expected response text.

If the user does not specify versions, use the defaults in this skill.

The generated project only needs to be runnable and testable. Unless the user asks for a specific domain, use simple sample behavior and lightweight naming.

## Docker Environment Contract

Use these defaults unless the user asks for something else.

### Gradle Wrapper Generation

Generate Gradle Wrapper through Docker:

```bash
docker run --rm \
  -v <project>:/workspace \
  -w /workspace \
  gradle:9.2.1-jdk25 \
  gradle wrapper --gradle-version 9.2.1
```

After wrapper generation, use only commands executed inside the Dev Container workflow.

Do not treat host execution as part of the supported workflow.

For example, the wrapper is generated through Docker, and validation happens through Dev Container:

```bash
devcontainer up --workspace-folder <project> --remove-existing-container
```

### Dev Container

Create `.devcontainer/devcontainer.json` with this shape:

```json
{
  "name": "project-name",
  "image": "mcr.microsoft.com/devcontainers/java:25",
  "remoteUser": "root",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspaces/${localWorkspaceFolderBasename},type=bind,consistency=cached",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "customizations": {
    "vscode": {
      "extensions": [
        "vscjava.vscode-java-pack",
        "vmware.vscode-boot-dev-pack",
        "vscjava.vscode-gradle"
      ],
      "settings": {
        "java.configuration.updateBuildConfiguration": "automatic"
      }
    }
  },
  "mounts": [
    "source=project-name-gradle-cache,target=/root/.gradle,type=volume"
  ],
  "postCreateCommand": "./gradlew --no-daemon test"
}
```

For a Spring Boot web app, add:

```json
"forwardPorts": [8080],
"portsAttributes": {
  "8080": {
    "label": "Spring Boot App",
    "onAutoForward": "notify"
  }
}
```

## Workflow

1. Confirm the target directory, project name, package name, and app type.
2. Create the project directory.
3. Create the initial project files:
   - `.gitignore`
   - `settings.gradle.kts`
   - `build.gradle.kts`
   - `README.md`
4. Generate Gradle Wrapper through `gradle:9.2.1-jdk25`.
5. If the user is in China or requests domestic mirrors, update `gradle/wrapper/gradle-wrapper.properties`:
   - `distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-9.2.1-bin.zip`
6. Create `.devcontainer/devcontainer.json`.
7. Add requested source code and tests.
   - For Spring Boot projects, the test template must use `@SpringBootTest`.
8. Validate through Dev Container CLI:
   - `devcontainer up --workspace-folder <project> --remove-existing-container`
9. For a web app, optionally start the app in the devcontainer and verify the endpoint with `curl`.

## Default Gradle Files

Default `settings.gradle.kts`:

```kotlin
rootProject.name = "project-name"
```

Default plain Java `build.gradle.kts`:

```kotlin
import org.gradle.jvm.toolchain.JavaLanguageVersion

plugins {
    java
}

group = "com.iokays"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}

repositories {
    maven("https://maven.aliyun.com/repository/public")
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:6.0.0")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test>().configureEach {
    useJUnitPlatform()
}
```

For Spring Boot 4 Web MVC projects, prefer this shape:

```kotlin
import org.gradle.api.tasks.testing.logging.TestExceptionFormat
import org.gradle.api.tasks.testing.logging.TestLogEvent
import org.gradle.jvm.toolchain.JavaLanguageVersion

plugins {
    java
    id("org.springframework.boot") version "4.0.6"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.iokays"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(25)
    }
}

repositories {
    maven("https://maven.aliyun.com/repository/public")
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-webmvc")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test>().configureEach {
    useJUnitPlatform()
    testLogging {
        events = setOf(TestLogEvent.PASSED, TestLogEvent.SKIPPED, TestLogEvent.FAILED)
        exceptionFormat = TestExceptionFormat.FULL
    }
}
```

## Validation

Validate the project by starting the Dev Container:

```bash
devcontainer up --workspace-folder <project> --remove-existing-container
```

The skill is complete only when the generated project is runnable through the Dev Container workflow and the tests pass in that workflow.

The validation succeeds when the configured `postCreateCommand` completes:

```bash
./gradlew --no-daemon test
```

For a Spring Boot endpoint, verify the app after tests pass:

```bash
docker exec <container-id> sh -lc 'cd /workspaces/<project> && ./gradlew --no-daemon bootRun'
docker exec <container-id> sh -lc 'curl -s http://localhost:8080/api/hello'
```

Stop the app after verification.

## README Contract

Generate a root `README.md` that explains:

- project name
- Java version
- Gradle Wrapper version
- Dev Container image
- how to start the Dev Container
- how to test inside the Dev Container
- how to run the app inside the Dev Container
- HTTP endpoint and expected response when applicable
- mirror settings when enabled

The README should make it clear that project execution is defined by the Dev Container contract.

## Rules

- Start from the Docker Dev Container environment contract.
- Do not initialize the project with `gradle init` on the host.
- Use the declared Docker images as the project environment.
- Keep the workflow focused on project creation, container configuration, and validation.
- Generate Gradle Wrapper through Docker.
- Use `./gradlew` after wrapper generation.
- Keep `.devcontainer/devcontainer.json` as a first-class project file.
- Prefer `workspaceMount` so projects work correctly inside larger Git repositories.
- Prefer a named Gradle cache volume mounted to `/root/.gradle`.
- For web apps, forward port `8080` unless the user requests another port.
- Keep the generated project small, editable, and easy to inspect.
- Treat Dev Container validation as the source of truth.

## Example User Request

```text
Use create-java-gradle-devcontainer-project to create a Java Gradle project in Docker Dev Container. Add a Spring Boot HTTP endpoint that returns "Hello from a Docker Dev Container." and generate a test for it.
```

## Expected Result

- Java Gradle Kotlin DSL project
- Gradle Wrapper generated through Docker
- `.devcontainer/devcontainer.json`
- application code when requested
- tests when requested
- root `README.md`
- successful Dev Container validation
