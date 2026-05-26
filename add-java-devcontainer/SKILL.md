---
name: add-java-devcontainer
description: Add or update a dev container for an existing Java project so development and build steps run inside Docker instead of relying on a local JDK. Use when Codex needs to create or refine `.devcontainer/devcontainer.json`, choose a Java devcontainer image, configure forwarded ports and cache mounts, and tailor `postCreateCommand` to Gradle or Maven projects.
---

# Add Java Devcontainer

## Overview

This skill adds a `.devcontainer` setup to an existing Java project so the project can build and run inside a container without depending on the local JDK.

Default to the Microsoft Java devcontainer image:

- `mcr.microsoft.com/devcontainers/java:25`

When the project uses Gradle, prefer a Gradle cache volume and use the project wrapper for validation.

## Workflow

1. Confirm the target project root.
2. Detect the build tool:
   - if `gradlew` exists, treat it as Gradle
   - if `pom.xml` exists, treat it as Maven
3. Read any existing `.devcontainer/devcontainer.json` before changing it.
4. Create or update `.devcontainer/devcontainer.json`.
5. Do not require a local JDK for development or validation steps.
6. For Java projects, default to:
   - `"image": "mcr.microsoft.com/devcontainers/java:25"`
   - `"remoteUser": "root"`
7. For Gradle projects, prefer:
   - a named volume mounted to `/root/.gradle`
   - `postCreateCommand` that uses `./gradlew`
8. For Maven projects, prefer:
   - a named volume mounted to `/root/.m2`
   - `postCreateCommand` that uses `mvn`
9. If the project is a web app, forward the app port and add `portsAttributes`.
10. If the user is in China or requests domestic mirrors:
    - update `gradle/wrapper/gradle-wrapper.properties` to a China mirror when appropriate
    - prefer `https://mirrors.cloud.tencent.com/gradle/` for Gradle wrapper downloads
11. Keep the configuration minimal and editable.
12. After editing, summarize how to reopen the project in the container and which command will validate inside the container.

## Gradle Default

For Gradle projects, use this shape unless the project needs something different:

```json
{
  "name": "java-devcontainer",
  "image": "mcr.microsoft.com/devcontainers/java:25",
  "remoteUser": "root",
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
    "source=project-gradle-cache,target=/root/.gradle,type=volume"
  ],
  "postCreateCommand": "./gradlew --no-daemon test"
}
```

Adjust these fields to the actual project:

- `name`
- `workspaceFolder`
- `forwardPorts`
- `portsAttributes`
- the Gradle cache volume name

## Port Guidance

- If the project exposes HTTP on port 8080, add:
  - `"forwardPorts": [8080]`
  - a `portsAttributes` label such as `"Spring Boot App"`
- If the project is not a web app, omit port forwarding by default.

## China Mirror Guidance

When the project uses Gradle wrapper in China-based environments, set:

- `gradle/wrapper/gradle-wrapper.properties`
- `distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-<version>-bin.zip`

Only update the wrapper URL when the project already uses Gradle wrapper or the user asks for it.

## Rules

- Do not switch the project back to local JDK assumptions.
- Prefer editing `.devcontainer/devcontainer.json` over inventing extra setup files.
- Keep `postCreateCommand` aligned with the detected build tool.
- For Gradle, use `./gradlew` instead of global `gradle`.
- Reuse the existing project port and build command when they are already established.
- Keep the resulting config short and easy to hand-edit.

## Assets

If a reusable starting point helps, copy from:

- `assets/devcontainer-gradle.json`

Then adapt it to the target project instead of pasting it blindly.
