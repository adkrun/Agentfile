---
title: "specification"
description: "This document defines the general Agent specification."
version: 0.1.0-draft
---

# Agent Specification

This document outlines the terminology, directory structure, and configuration standards used to build and define an Agent.

## Glossary

| Term | Definition |
| :--- | :--- |
| **Agent** | **The root orchestrator.** Defines the high-level mission, manages global configurations (like model parameters), and dictates which skills are accessible. |
| **Identity** | **The external persona.** Defines the voice, tone, personality traits, and exactly "how" the agent communicates with the outside world. |
| **Memory** | **The contextual ledger.** Stores past interactions, learned facts, and state. This enables the agent to maintain continuity and recall information across sessions. |
| **Prompt** | **The compiled instruction set.** Dynamically combines directives from the Agent, Identity, Soul, and relevant Skills—alongside the user's query—to send to the underlying LLM. |
| **Skill** | **A domain-specific module.** Contains localized instructions and a subset of executable **Tools** dedicated to a specific domain (e.g., "Social Media", "Trading"). |
| **Soul** | **The internal compass.** Defines the core values, ethical boundaries, motivations, and the "why" behind the agent's existence and decision-making. |
| **Tool** | **An executable function.** An executable script (e.g., written in Go or TypeScript) with defined input and output schemas, used by a Skill to interact with external systems. |
| **User** | **The user profile.** Contains the specific knowledge, preferences, background, and context *about* the human interacting with the agent. |

## Directory Hierarchy

```text
<agent-name>/
├── AGENT.md                # Core configuration & high-level mission
├── IDENTITY.md             # Personality, voice, and external traits
├── SOUL.md                 # Core values, motivations, and internal logic
├── memories/               # Directory containing session-based context
│   └── <session-name>/     # Unique identifier for the interaction thread
│       └── MEMORY.json     # Stored state, facts, and interaction logs
├── prompts/                # Directory containing custom prompt templates
│   └── <prompt-name>/      
│       └── PROMPT.md       # Logic for a specific prompt
└── skills/                 # Domain-specific capabilities
    ├── <skill-name>/       # E.g., "social", "trading"
    │   ├── SKILL.md        # Instructions for this skill
    │   └── scripts/        # Executable tools belonging to this skill
    │       ├── calculator.ts 
    │       ├── <tool-name>.ts
    │
    └── <another-skill>/    # Additional skills follow the exact same pattern
        ├── SKILL.md
        └── scripts/
            ├── <tool-name>.go
```

## Agent Anatomy

An Agent component can be defined using a Markdown (`.md`), Go (`.go`), TypeScript (`.ts`), or JSON (`.json`) file. The core configuration for that component is stored inside the **YAML Frontmatter** at the very top of the file, followed by the specific content, instructions, or code.

```markdown
---
name: example-component
description: A brief description of what this does.
version: 1.0.0
---

<Markdown content / Tool code content / Memory content>
```

### YAML Frontmatter Fields (required)

These fields are standard across all component definition files.

| Field | Type | Required | Constraints |
| :--- | :--- | :--- | :--- |
| `name` | string | Yes | **Max:** 64 characters.<br>**Format:** Lowercase letters, numbers, dots (`.`), and hyphens (`-`) only.<br>**Rule:** Cannot start or end with a dot or hyphen. |
| `description` | string | Yes | **Max:** 1024 characters.<br>**Rule:** Must be non-empty. Clearly describes what the component does and when to use it. |
| `version` | string | No | The semantic version of the component (e.g., `0.1.0`). |
| `license` | string | No | The license name (e.g., `MIT`) or a reference to a bundled license file. |
| `keywords` | string[] | No | A list of relevant keywords associated with the component (e.g., `["social", "lead"]`). |


#### `name` field

The required `name` field:
- Must be 1-64 characters
- May only contain unicode lowercase alphanumeric characters, dots, and hyphens (`a-z`, `0-9`, `.`, and `-`)
- Must not start or end with a hyphen (`-`) or dot (`.`)
- Must not contain consecutive hyphens (`--`) or consecutive dots (`..`)
- Must match the parent directory name

Valid examples:
```yaml
name: pdf-processing
```
```yaml
name: data.analysis
```
```yaml
name: v1.code-review
```

Invalid examples:
```yaml
name: PDF-Processing  # uppercase not allowed
```
```yaml
name: .pdf            # cannot start with a dot
```
```yaml
name: -data           # cannot start with a hyphen
```
```yaml
name: pdf..processing # consecutive dots not allowed
```

#### `description` field

The required `description` field:
- Must be 1-1024 characters
- Should describe both what the skill does and when to use it
- Should include specific keywords that help agents identify relevant tasks

Good example:
```yaml
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
```

Poor example:
```yaml
description: Helps with PDFs.
```

#### `version` field

The optional `version` field:
- Should follow Semantic Versioning (SemVer) formatting (e.g., `MAJOR.MINOR.PATCH`)
- Helps developers and users track updates, bug fixes, and compatibility 

Examples:
```yaml
version: 1.0.0
```
```yaml
version: 0.1.0-beta
```

#### `license` field

The optional `license` field:
- Specifies the license applied to the Agent component
- We recommend keeping it short (either the name of a standard license or the name of a bundled license file)

Good example:
```yaml
license: MIT
```

Poor example:
```yaml
license: Proprietary. LICENSE.md has complete terms
```

#### `keywords` field

The optional `keywords` field:
- Must be formatted as an array of strings
- Helps categorize the component and improves searchability/discovery for both users and orchestrating agents
- Recommended to use lowercase words or short, relevant phrases

Example:
```yaml
keywords: ["pdf", "document parsing", "data extraction", "forms"]
```

### Body content (required)
The body after the frontmatter contains the core instructions (for `.md` files) or executable logic (for `.go` and `.ts` files). There are no format restrictions. Write whatever helps the agent understand the component's purpose, adopt the necessary persona, or execute tasks effectively.

Recommended inclusions (depending on the component type):
- Step-by-step instructions or logic.
- Examples of inputs and outputs.
- Guidelines or logic for handling common edge cases.

#### Variables
To ensure the Agent can dynamically inject context into components, use the syntax `{{name|description}}`. Variables are supported within the body content of **`.md` files only**.

| Field | Required | Definition | Constraints |
| :--- | :--- | :--- | :--- |
| `name` | Yes | The unique key used by the orchestrator to resolve and inject the value. | **Max:** 32 characters.<br>**Format:** Lowercase letters, numbers, and hyphens (`-`) only.<br>**Rule:** Must be non-empty. |
| `description` | Yes | A human-readable note explaining the purpose, constraints, or expected data type. | **Max:** 256 characters.<br>**Rule:** Must be non-empty. |

#### Syntax Rules
* **Delimiter:** Variables must be wrapped in double curly braces: `{{...}}`.
* **Separator:** The `name` and `description` must be separated by a single pipe character (`|`).
* **Placement:** Variables are processed only within the body content (after the YAML frontmatter).
* **Scope:** Variables are supported exclusively in `.md` files. Logic files (e.g., `.ts`, `.go`) must handle variable injection through their respective runtime environments.

#### Usage Example

```markdown
---
name: interaction-summary
description: Generates a contextual summary for the current session.
---

Hello, I am {{agent-name|The current agent's name}}. 
I have been observing our interaction regarding {{topic|The current subject of discussion}}.
Based on our history, the user {{user-name|The name of the human}} prefers {{preference|The user's specific interaction style}}.
```

## AGENT.md

## IDENTITY.md

## MEMORY.json

## PROMPT.md

## SKILL.md

## SOUL.md

## TOOL

## USER.md