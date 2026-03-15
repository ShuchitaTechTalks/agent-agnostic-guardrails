# JavaOne Agent Demo - Setup and Execution Guide

This guide is for developers running the learning session on cross-agent quality guardrails for Spring Boot code generation.

The session objective is to show how a reusable `AGENTS.md` pattern improves code quality across agent platforms by combining:

- Context-aware generation instructions (`AGENTS.md` at repo and package scope)
- Verifiable static analysis (`Checkstyle`,  `SpotBugs`)


---

## Prerequisites

- Java 21 JDK - [Eclipse Adoptium](https://adoptium.net/) (recommended) or [Oracle](https://www.oracle.com/java/technologies/downloads/#java21)
- Maven 3.9+ - [maven.apache.org](https://maven.apache.org/download.cgi)
- One IDE: [VS Code](https://code.visualstudio.com) or [IntelliJ IDEA](https://www.jetbrains.com/idea/)
- GitHub account with Copilot access - [github.com/features/copilot](https://github.com/features/copilot)

Verify local tooling:

```powershell
java -version
mvn -v
```

---

## Part 1: IDE and AI Plugin Setup

### 1.1 VS Code Setup

1. Install **GitHub Copilot** from Extensions (`Ctrl+Shift+X`).
2. Sign in to GitHub when prompted.
3. Install **Junie** extension (if your org/session uses it).
4. Confirm chat is available (`Ctrl+Shift+I`) and inline assist works (`Ctrl+I`).

### 1.2 IntelliJ IDEA Setup

1. Open **Settings -> Plugins -> Marketplace**.
2. Install **GitHub Copilot** plugin.
3. Install **Junie** plugin (if available to your JetBrains account/org).
4. Restart IDEA.
5. Sign in to GitHub Copilot from the plugin prompt.
6. Verify both assistants are enabled in the IDE tool window/chat panel.

---

## Part 2: Generate the Spring Boot Project

In your AI chat (Copilot/Junie), ask it to generate the project using:

- `spring-boot-project-generation/project-generation-prompt.md`

Recommended prompt:

```text
Generate a Spring Boot project in the current folder based strictly on the instructions in spring-boot-project-generation/project-generation-prompt.md.
```

After generation, validate that the expected module/package structure exists before moving to Step 2.

---

## Part 3: Step 2 - Add AGENTS.md Guardrails

Copy instruction templates from `agentfiles/` into the generated project as `AGENTS.md` files.

| Source file | Destination |
|---|---|
| `agentfiles/PARENT-AGENTS.md` | `<project-root>/AGENTS.md` |
| `agentfiles/CONTROLLER-AGENTS.md` | `<project-root>/src/main/java/.../controller/AGENTS.md` |
| `agentfiles/SERVICE-AGENTS.md` | `<project-root>/src/main/java/.../service/AGENTS.md` |
| `agentfiles/REPO-AGENTS.md` | `<project-root>/src/main/java/.../repository/AGENTS.md` (or `.../repo/AGENTS.md` if that is your package name) |

Why this matters:

- Parent-level `AGENTS.md` defines repo-wide standards.
- Package-level `AGENTS.md` files provide role-specific instructions for controller, service, and data-access code.
- The pattern is agent-agnostic and reusable across chat sessions.

---

## Part 4: Build, Analyze, and Measure

Run build and quality checks from project root:

```powershell
mvn clean test
mvn verify
```

Use outputs to compare two runs in your demo:

1. Baseline generation (without `AGENTS.md` guardrails)
2. Guardrailed generation (with `AGENTS.md` files in place)

Track at minimum:

- Total static-analysis violations (`Checkstyle`,  `SpotBugs`)
- Unit test coverage

---

## Part 5: Suggested Live Demo Flow

1. Generate project without AGENTS guardrails.
2. Run `mvn verify` and capture violations + coverage snapshot.
3. Add `AGENTS.md` files as documented in Part 3.
4. Regenerate or refine code with the same functional requirements.
5. Re-run `mvn verify` and compare metrics.
6. Present observed deltas (session case study target: major violation reduction and coverage improvement).

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `java` not found | Install Java 21 JDK, restart terminal, re-run `java -version` |
| `mvn` not found | Install Maven, add Maven `bin` to `PATH`, re-run `mvn -v` |
| Build fails on Java version | Confirm project `pom.xml` uses Java 21 and local JDK is 21 |
| Copilot not responding | Re-check GitHub sign-in in IDE; reload/restart IDE |
| Junie unavailable | Confirm plugin availability/licensing in your org and re-authenticate |
| Missing generated files | Re-run generation prompt and reference `project-generation-prompt.md` explicitly |

---

## Reference Files in This Repo

- `spring-boot-project-generation/project-generation-prompt.md` - Spring Boot generation instructions
- `agentfiles/PARENT-AGENTS.md` - repo-level guardrails template
- `agentfiles/CONTROLLER-AGENTS.md` - controller guardrails template
- `agentfiles/SERVICE-AGENTS.md` - service guardrails template
- `agentfiles/REPO-AGENTS.md` - repository guardrails template
- `Agent-Agnostic-Guardrails.pdf` - presentation in PDF format
- `presentationascii.adoc` ASCII-compatible presentation format
- `presentationmarkup.md` markup-compatible presentation format
- `presentationascii.adoc` ASCII-compatible presentation format
- `presentationmarkup.md` markup-compatible presentation format
