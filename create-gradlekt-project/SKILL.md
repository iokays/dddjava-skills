---
name: create-gradlekt-project
description: Create Java Gradle projects that use Kotlin DSL. Use when Codex needs to verify the local environment with `java --version` and `gradle --version`, initialize an empty project with `gradle init`, and then use the project's own `gradlew` for later build and test tasks. Default to JDK 25 and Gradle 9 unless the user explicitly asks for different versions.
---

# Create Gradlekt Project

## Overview

Check the local Java and Gradle environment first, use local `gradle` to initialize an empty project, then switch to the project's own `gradlew` for later work.

If the skill contains a matching local Gradle distribution zip, prefer that local archive instead of downloading the same Gradle version from the network.

## Workflow

1. Confirm the target directory and project name.
2. Run `java --version`.
3. Run `gradle --version`.
4. Default to JDK 25 and Gradle 9 when the user does not specify versions.
5. If the user specifies a Java or Gradle version, honor that version in the generated files.
6. If JDK is missing, tell the user to install it from:
   - `https://www.oracle.com/java/technologies/downloads/`
7. If Gradle is missing, tell the user to install it from:
   - international: `https://gradle.org/releases/`
   - China: `https://mirrors.cloud.tencent.com/gradle/`
8. After installation guidance, explain how to configure environment variables for the current operating system and shell.
9. Ensure `java --version` matches the requested Java major version.
10. Ensure `gradle --version` runs successfully and is compatible with the requested Gradle major version when the user explicitly requires one.
11. Check whether the skill already contains a matching local Gradle distribution under `assets/distributions/`.
12. If a matching local Gradle zip exists, prefer local reuse and avoid network download for that Gradle version.
13. Run `gradle init` to create an empty project.
14. After the project is created, if the user is in China or the project targets a China-based environment, update `gradle/wrapper/gradle-wrapper.properties` so `distributionUrl` uses `https://mirrors.cloud.tencent.com/gradle/`.
15. If the user is in China or the project targets a China-based environment, add the Aliyun Maven mirror when configuring repositories.
16. Generate a project-level `README.md` so the project can be understood and run without referring back to the skill.
17. After the project is created, use `./gradlew` for build, test, and later Gradle tasks instead of the local global `gradle`.
18. Verify the empty project with `./gradlew clean build`.
19. After verification succeeds, run `./gradlew clean` to remove build artifacts from the empty project.
20. Apply the user-provided project architecture on top of the initialized project when needed.

## Quick Start

Check the environment:

```bash
java --version
gradle --version
```

If a command is missing, do not continue directly to project creation. First tell the user what to install and how to configure the environment for the current system.

Then create the project:

```bash
gradle init \
  --type java-application \
  --dsl kotlin \
  --test-framework junit-jupiter \
  --package com.iokays \
  --project-name my-app \
  --java-version 25 \
  --overwrite \
  --incubating \
  --use-defaults
```

After initialization:

1. Enter the project directory.
2. Use `./gradlew` instead of the local global `gradle`.
3. If the skill contains a matching local Gradle zip, reuse it as the preferred local Gradle distribution source.
4. Replace or adjust the generated `build.gradle.kts`, `gradle.properties`, and `settings.gradle.kts` with the required architecture.
5. If the user is in China, update `gradle/wrapper/gradle-wrapper.properties` to the China mirror and add the Aliyun Maven mirror to repositories when needed.
6. If the user requested other versions, adjust the Java version and `gradle-wrapper.properties` before finishing.
7. Generate `README.md` in the project root.
8. Run `./gradlew clean build` to verify the empty project.
9. Run `./gradlew clean` so the empty project does not keep build artifacts by default.
10. Create each module directory and put a `build.gradle.kts` inside it.

## Initialization Result

- an empty Java application project initialized by `gradle init`
- generated `gradlew` and `gradlew.bat`
- `gradle/wrapper/gradle-wrapper.properties`
- a root `README.md`
- initial Kotlin DSL Gradle files that can be adjusted to the required architecture

## Rules

- Default to JDK 25.
- Default to Gradle 9.
- Allow the user to override the default Java and Gradle versions.
- Fail fast when `java --version` does not report the required Java version.
- Fail fast when `gradle --version` cannot run.
- When JDK is missing, point the user to `https://www.oracle.com/java/technologies/downloads/`.
- When Gradle is missing, point the user to `https://gradle.org/releases/` or `https://mirrors.cloud.tencent.com/gradle/`.
- Explain environment variable setup using the current operating system and shell instead of giving generic instructions only.
- Use local global `gradle` only to initialize the project.
- If `assets/distributions/` contains the required Gradle zip, prefer that local archive over downloading the same version again.
- When the user is in China, prefer `https://mirrors.cloud.tencent.com/gradle/` in `gradle-wrapper.properties`.
- When the user is in China, allow adding the Aliyun Maven mirror in repositories.
- Always generate a root `README.md` for the created project.
- After initialization, use `./gradlew` for build, test, and later Gradle tasks.
- Treat `./gradlew clean build` as the default validation step for the empty project.
- After validation, run `./gradlew clean` to leave the empty project in a clean state.
- Keep `--enable-preview` enabled for compile, exec, and test tasks.
- Keep tests disabled by default unless `-PrunTests` or an explicit test task is requested.
- Update `distributionUrl` in `gradle-wrapper.properties` when the user requests a different Gradle version.

## Missing Environment Guidance

Use this response style when tools are missing:

- If `java --version` fails:
  - tell the user JDK is not installed or not configured
  - recommend Oracle download page: `https://www.oracle.com/java/technologies/downloads/`
- If `gradle --version` fails:
  - tell the user Gradle is not installed or not configured
  - recommend Gradle releases: `https://gradle.org/releases/`
  - for users in China, also recommend: `https://mirrors.cloud.tencent.com/gradle/`
  - if the error contains `Failed to load native library 'libnative-platform.dylib' for Mac OS X aarch64` while using Xcode on macOS, tell the user to give Xcode Full Disk Access or full access permission, then retry `gradle --version`

For environment variables, tailor the guidance:

- macOS with `zsh`:
  - edit `~/.zshrc`
  - example JDK:
    - `export JAVA_HOME=/path/to/jdk`
    - `export PATH="$JAVA_HOME/bin:$PATH"`
  - example Gradle:
    - `export GRADLE_HOME=/path/to/gradle`
    - `export PATH="$GRADLE_HOME/bin:$PATH"`
  - then run `source ~/.zshrc`
- Linux with `bash`:
  - edit `~/.bashrc`
  - add the same `JAVA_HOME`, `GRADLE_HOME`, and `PATH` exports
  - then run `source ~/.bashrc`
- Windows:
  - tell the user to configure `JAVA_HOME` and `GRADLE_HOME` in System Environment Variables
  - add `%JAVA_HOME%\bin` and `%GRADLE_HOME%\bin` to `Path`

## References

Keep the skill minimal until the user provides the target project architecture.

Local distribution archive currently present:

- `assets/distributions/gradle-9.2.1-bin.zip`

## README Contract

The generated project `README.md` should be short but sufficient for standalone use.

Include at least:

- project name
- required Java version
- required Gradle usage rule: prefer `./gradlew`
- how to build: `./gradlew clean build`
- how to test: `./gradlew test`
- how to clean: `./gradlew clean`
- note when China mirror or repository mirror is enabled

Do not require the reader to open the skill just to understand how to run the generated project.
