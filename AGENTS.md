# AGENTS.md

## Purpose

This repository stores Codex skills for Java and Dev Container oriented project scaffolding.

The main goal of work in this repository is to create, refine, and validate skill definitions that help Codex generate runnable Java projects and related developer setup with predictable behavior.

## Scope

Agents working in this repository should focus on:

- creating or updating skill documentation in `SKILL.md`
- maintaining companion metadata under `agents/`
- keeping each skill small, explicit, and easy to execute
- preserving a Docker-first workflow for Dev Container related skills
- adding or refining sample projects only when the user explicitly asks for them

Agents should not introduce unrelated repository restructuring, broad cleanup, or speculative abstractions unless the user asks for that work.

## Repository Layout

- `add-java-devcontainer/`
  Adds or updates a Java project's Dev Container configuration.
- `create-gradlekt-project/`
  Creates Java Gradle Kotlin DSL projects using a host-initialized workflow.
- `create-java-gradle-devcontainer-project/`
  Creates runnable Java Gradle projects using Docker Dev Container as the source of truth.
- `samples/`
  Holds generated or example projects when the user explicitly wants sample output.

Each skill directory should normally contain:

- `SKILL.md`
- `agents/openai.yaml`
- optional `assets/`
- optional `scripts/`

## Source Of Truth

For each skill, the primary source of truth is the skill document itself:

- behavior belongs in `SKILL.md`
- lightweight interface metadata belongs in `agents/openai.yaml`
- reusable templates or starter files belong in `assets/`

If behavior changes, update `SKILL.md` first so the repository documentation stays aligned with the intended workflow.

## Editing Rules

- Prefer minimal, targeted edits.
- Preserve the existing tone: direct, operational, and constraint-oriented.
- Keep defaults explicit. Do not rely on hidden assumptions when a rule can be stated plainly.
- When tightening a workflow, encode the rule in the skill instead of leaving it to prompting alone.
- Do not remove user-authored or unrelated local changes unless explicitly requested.
- Use ASCII by default unless a file already requires another character set.

## Skill Authoring Guidelines

When creating or updating a skill in this repository:

1. Start with a short description of what the skill does.
2. State hard constraints explicitly when the workflow must not drift.
3. Keep required inputs small and practical.
4. Prefer deterministic commands and validation steps.
5. Separate defaults from optional behavior.
6. Include completion criteria so agents can tell when the work is actually done.
7. If a workflow depends on Docker, Dev Container, Java, Gradle, or network mirrors, say so directly.

## Dev Container Skill Policy

For `create-java-gradle-devcontainer-project`, the repository expects these rules to remain enforced unless the user requests a change:

- Docker Dev Container is the only supported execution model.
- Validation must happen through the Dev Container workflow.
- Docker-generated Gradle Wrapper is part of the expected setup.
- For Spring Boot projects, tests must use native `@SpringBootTest` style.
- The skill's job is to generate a runnable project; the specific business domain is secondary.

Do not weaken these constraints silently.

## Validation Expectations

When you change a skill, validate at the right level for the request:

- for documentation-only updates, verify consistency across `SKILL.md` and `agents/openai.yaml`
- for workflow changes, prefer executing the workflow described by the skill when practical
- for Dev Container project generation, treat `devcontainer up` and its configured validation command as the source of truth

If validation cannot be run, say exactly what was not verified and why.

## Samples Policy

- Only create or modify files under `samples/` when the user explicitly asks for a sample project.
- Keep generated sample projects isolated under their own directory.
- Do not commit sample build artifacts, local caches, or transient output unless the user explicitly requests them.

## Commit Guidance

- Keep commits focused on one logical change.
- Prefer commit messages that describe the repository-level outcome, for example:
  - `Add Java Gradle devcontainer project skill`
  - `Tighten Spring Boot testing rules in devcontainer skill`
  - `Document Docker-only validation policy for Java skills`

## Response Guidance For Agents

- Be explicit about assumptions when the user leaves project details open.
- Ask follow-up questions only when the decision has real downstream impact.
- If the skill already defines a hard constraint, follow it instead of improvising a parallel workflow.
- Summaries should emphasize what changed, what was validated, and any remaining limits.

## When In Doubt

- Prefer clarity over cleverness.
- Prefer repository conventions over ad hoc variation.
- Prefer codified rules in the skill over one-off prompt behavior.
