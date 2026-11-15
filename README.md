# Unknown

> **The memory layer for AI-generated software.**
>
> Unknown preserves the engineering intent behind code
> changes---connecting code to the original goals, constraints,
> decisions, and development context---so future engineers can safely
> understand and evolve software.

[![Status](https://img.shields.io/badge/status-early%20design-blue)](#project-status)
[![License](https://img.shields.io/badge/license-TBD-lightgrey)](#project-status)

------------------------------------------------------------------------

## Table of Contents

-   [The Problem](#the-problem)
-   [The Solution](#the-solution)
-   [Core Principles](#core-principles)
-   [How It Works](#how-it-works)
-   [Core Capabilities](#core-capabilities)
-   [The `.unknown/` Intent Manifest](#the-unknown-intent-manifest)
-   [Architecture](#architecture)
-   [Technology Stack](#technology-stack)
-   [Development Roadmap](#development-roadmap)
-   [Proposed CLI](#proposed-cli)
-   [Privacy and Trust](#privacy-and-trust)
-   [Project Status](#project-status)
-   [Contributing](#contributing)
-   [License](#license)

------------------------------------------------------------------------

## The Problem

Software teams already have mature systems for understanding the history
of a codebase:

  System                       What it tells us
  ---------------------------- -----------------------------------------
  Git                          What changed?
  Pull requests                Who reviewed the change?
  Issue trackers               What problem was being solved?
  Architecture documentation   What architecture is intended to exist?

AI-assisted development introduces a gap:

**Why was this code written this way?**

A developer may spend significant time guiding an AI coding agent:

1.  Implement caching.
2.  Do not introduce external infrastructure.
3.  Support consistency across multiple instances.
4.  Optimize for high traffic.

The final commit records the resulting code, but often loses the
decisions and constraints that shaped it.

Months later, another engineer may see a seemingly complicated
implementation and have no reliable way to discover:

-   Why this architecture was selected.
-   Which constraints influenced the implementation.
-   Which alternatives were considered or rejected.
-   Which assumptions must remain true.
-   What should not be broken during a refactor.

The code survives. The engineering context often does not.

## The Solution

Unknown adds an **intent layer** to a software repository.

Instead of preserving only:

``` text
Code
  |
Commit
```

Unknown aims to preserve:

``` text
Code
  |
Intent
  |
Engineering decisions
  |
Constraints and trade-offs
  |
Relevant development context
```

Unknown is not intended to preserve private model reasoning or hidden
chain-of-thought. Its purpose is to capture **human engineering
intent**: goals, explicit constraints, decisions, trade-offs, and
relevant context that can be reviewed and maintained by engineers.

## Core Principles

-   **Intent belongs with the code.** Keep repository-level intent
    versionable and reviewable.
-   **Human intent is the source of truth.** AI-generated summaries must
    be inspectable and correctable.
-   **Do not block development.** Missing metadata should not prevent
    normal editing or committing.
-   **Identity must survive refactoring.** Avoid relying only on file
    paths and line numbers.
-   **Be incremental.** Start with a local repository workflow before
    introducing centralized infrastructure.
-   **Respect developer privacy.** Capture only the context the
    developer or team authorizes.

## How It Works

At a high level, the planned workflow is:

1.  A developer works in an IDE and makes a code change, potentially
    with an AI coding assistant.
2.  The Unknown extension gathers authorized inputs such as the selected
    code, Git diff, developer-provided instructions, and relevant
    repository context.
3.  The intent extraction engine proposes a structured description of
    the change.
4.  The developer reviews and edits the proposed purpose, constraints,
    and decisions.
5.  Unknown stores the approved intent in the repository's `.unknown/`
    directory.
6.  Code intelligence associates the intent with structural code
    components rather than fragile line-number references.
7.  Later, an engineer can inspect the intent or use it to reconstruct
    useful context for a new task.

``` text
Developer
    |
IDE / AI coding environment
    |
Unknown IDE extension
    |
    +--------------------------+
    |                          |
Intent extraction        Code intelligence
    |                          |
    +------------+-------------+
                 |
          Intent manifests
                 |
            .unknown/
                 |
          Git repository
                 |
       Context reconstruction
                 |
          IDE intent panel
```

## Core Capabilities

The capabilities below describe the intended product direction. They are
not a claim that every component is already implemented.

### 1. Intent Manifest System

A repository-local `.unknown/` directory stores structured intent
metadata for code components. The format is intended to be versioned,
validated, and suitable for Git.

### 2. Code Intelligence

Parse source code into structural representations such as functions,
classes, interfaces, and dependencies. Planned language tooling includes
Tree-sitter, the Go AST, and Clang/LLVM tooling.

### 3. Structural Component Identity

File paths and line numbers change during ordinary development. Unknown
aims to maintain component identity across moves, renames, and refactors
using normalized structural representations and fingerprints.

Similarity matching should be treated as evidence, not certainty:
ambiguous matches should be surfaced for review rather than silently
attaching intent to the wrong component.

### 4. Intent Extraction

Generate structured intent proposals from authorized inputs, potentially
including:

-   Developer instructions and corrections.
-   Git diffs and changed files.
-   Relevant repository context.
-   Existing architecture documentation.
-   Developer-provided comments or notes.

Expected output includes purpose, constraints, design decisions,
trade-offs, risks, and future considerations. Developers should be able
to review and edit generated content before it is saved.

### 5. IDE Understanding Interface

An IDE panel should show the intent associated with the selected code
component, including:

-   Purpose.
-   Creation or update information.
-   Important constraints.
-   Design decisions and their rationale.
-   Related commits and documentation.
-   An action to continue with the original context.

### 6. Context Reconstruction

A future engineer should be able to request the relevant context for a
component. Unknown can assemble available information such as:

-   Current repository state.
-   Relevant source files.
-   Intent manifests.
-   Related decisions and constraints.
-   Relevant Git history and documentation.

That context can then be supplied to an AI assistant as task context.
The reconstructed context should be traceable to its sources and should
not be presented as a verbatim record of an unavailable conversation.

### 7. Git and Pull Request Integration

Unknown is designed to complement Git rather than replace it.

Potential workflows include:

-   Detecting changed components.
-   Offering to generate or update intent metadata.
-   Reporting intent coverage in a pull request.
-   Highlighting changed components with missing or stale intent.
-   Linking intent to commits and pull requests.

Missing intent should be a warning or review signal---not a default
commit blocker.

### 8. Repository Memory and Semantic Search

A later centralized layer may index repositories, components, intents,
constraints, decisions, and relationships. Semantic search could support
questions such as:

-   Where is retry logic implemented?
-   Which components relate to payment reliability?
-   Which decisions were made to satisfy a compliance constraint?

The initial repository-local manifest remains the foundation;
centralized storage is a later capability.

### 9. Engineering Knowledge Graph

The long-term vision connects code components with intent, decisions,
constraints, commits, pull requests, issues, architecture decision
records, incidents, and documentation.

Example relationship types include:

-   `implements`
-   `depends_on`
-   `created_by`
-   `related_to`
-   `documents`
-   `violates`

### 10. Architectural Discovery and Invariants

Future analysis may discover module boundaries, dependency
relationships, call graphs, and architectural layers. Teams could define
or review invariants such as:

-   Payment code must not depend on notification delivery.
-   Repository-layer code is responsible for database access.
-   Personally identifiable information must not be written to logs.

Potential violations should be explained with evidence and offered for
developer review.

### 11. Multi-Provider AI Service

The planned AI layer is provider-agnostic, with potential integrations
for OpenAI, Gemini, and Anthropic models. Its responsibilities include
intent generation, intent updates, context reconstruction, and code
explanation.

Provider support and exact integration contracts remain to be designed.

### 12. CLI and Metrics

A future CLI may support repository initialization, scanning, intent
generation, updates, search, and diagnostics. Metrics may include intent
coverage, components with missing intent, and AI-assisted change
coverage where that information is available and authorized.

------------------------------------------------------------------------

## The `.unknown/` Intent Manifest

A proposed repository layout:

``` text
project/
├── src/
│   └── payment/
│       └── retry.go
├── .unknown/
│   └── payment.retry.intent.json
└── ...
```

### Example manifest

The following is an illustrative format, not yet a finalized schema:

``` json
{
  "schema_version": "0.1",
  "component": {
    "path": "src/payment/retry.go",
    "identity": "component-id-placeholder"
  },
  "created": {
    "type": "AI-assisted",
    "date": "2026-07-12"
  },
  "purpose": {
    "summary": "Implement payment retry handling without duplicate transactions"
  },
  "constraints": [
    "Must support multi-region deployment",
    "Cannot change the database schema",
    "Must maintain idempotency"
  ],
  "design_decisions": [
    {
      "decision": "Use exponential backoff",
      "reason": "Avoid overwhelming the payment provider during outages"
    }
  ],
  "related_commits": [
    "a82f91"
  ]
}
```

### Manifest design goals

The manifest specification should define:

-   Schema versioning and compatibility.
-   Stable component identifiers and identity history.
-   Required and optional fields.
-   Validation and migration behavior.
-   How multiple components and files are represented.
-   How intent changes are reviewed and merged.
-   How deleted, split, and merged components are handled.
-   Provenance for generated versus human-authored fields.

The manifest should remain readable in ordinary code review and should
not require a hosted service to be useful.

------------------------------------------------------------------------

## Architecture

### Proposed logical architecture

``` text
                 Developer
                     |
       IDE / AI coding environment
                     |
           Unknown IDE extension
                     |
          +----------+----------+
          |                     |
  Intent extraction      Code intelligence
          |                     |
          |               Parser / AST
          |                     |
          |             Structural identity
          |                     |
          +----------+----------+
                     |
              Repository layer
                     |
                  .unknown/
                     |
                Git repository
                     |
          +----------+----------+
          |                     |
  Context reconstruction   Repository memory
          |                     |
          |              Search / knowledge graph
          |                     |
          +----------+----------+
                     |
              IDE intent panel
```

### Proposed service architecture

The longer-term technical direction separates the IDE experience,
backend orchestration, code analysis, AI integration, and persistence:

``` text
VS Code Extension (TypeScript)
            |
       REST / gRPC
            |
       Go backend
            |
   +--------+---------+
   |                  |
Code analysis       AI service
(Tree-sitter,       (provider
Go AST, Clang)       adapters)
   |                  |
   +--------+---------+
            |
   Repository / storage layer
            |
   PostgreSQL + pgvector
            |
   Search and knowledge graph
```

The initial implementation should favor a local-first workflow and avoid
introducing distributed services before the product needs them.

------------------------------------------------------------------------

## Technology Stack

The following is the proposed stack, subject to validation during
implementation.

  -----------------------------------------------------------------------
  Layer                               Proposed technologies
  ----------------------------------- -----------------------------------
  IDE extension                       TypeScript, VS Code Extension API,
                                      Webview API, Language Server
                                      Protocol

  Backend                             Go, REST, gRPC

  Code intelligence                   Tree-sitter, Go AST, Clang AST,
                                      LLVM

  AI integration                      OpenAI API, Gemini API, Anthropic
                                      API

  Persistence                         `.unknown/` manifests initially;
                                      PostgreSQL and pgvector later

  Caching                             Redis, if required by later
                                      workloads

  Infrastructure                      Docker, GitHub Actions, Terraform;
                                      Kubernetes later

  Observability                       OpenTelemetry
  -----------------------------------------------------------------------

### Architectural rationale

-   **TypeScript** provides direct access to the VS Code extension
    ecosystem.
-   **Go** is a candidate for backend services and repository-oriented
    tooling.
-   **Tree-sitter** offers a path toward multi-language syntax parsing.
-   **Go AST** provides native structural analysis for Go code.
-   **Clang/LLVM** can support deeper C and C++ analysis.
-   **PostgreSQL and pgvector** are candidates for later structured and
    semantic retrieval.
-   **Local manifests** keep the first useful version independent of a
    hosted database.

------------------------------------------------------------------------

## Development Roadmap

The roadmap is intentionally incremental. The complete vision should not
be treated as the first release scope.

### Phase 1 --- Repository-local MVP

Target direction: first usable release.

-   [ ] VS Code extension scaffold.
-   [ ] Select a code component and request an intent summary.
-   [ ] Generate a structured intent proposal.
-   [ ] Allow the developer to review and edit the proposal.
-   [ ] Create and update `.unknown/` manifests.
-   [ ] Add initial AST parsing and component fingerprinting.
-   [ ] Display intent in an IDE panel.

### Phase 2 --- Git and discoverability

-   [ ] Git change detection and component association.
-   [ ] Intent coverage reporting.
-   [ ] Pull request integration.
-   [ ] Intent search.
-   [ ] Manifest validation and migration support.

### Phase 3 --- Context reconstruction

-   [ ] Retrieve relevant manifests and source context.
-   [ ] Connect intent to commits and documentation.
-   [ ] Build reproducible context packages.
-   [ ] Integrate reconstructed context with supported AI assistants.

### Phase 4 --- Repository intelligence

-   [ ] PostgreSQL-backed repository memory.
-   [ ] Semantic search with pgvector.
-   [ ] Knowledge graph and relationship traversal.
-   [ ] Architecture and dependency discovery.
-   [ ] Architectural invariant checks.

### Phase 5 --- Team and enterprise capabilities

-   [ ] Multi-repository support.
-   [ ] Enterprise deployment options.
-   [ ] Team-level governance and access controls.
-   [ ] Expanded language support.
-   [ ] Operational metrics and observability.

------------------------------------------------------------------------

## Proposed CLI

The following commands are examples of the intended CLI interface. They
are not necessarily available in the current project.

``` bash
# Initialize Unknown metadata in a repository
unknown init

# Scan repository structure
unknown scan

# Generate intent for selected or changed components
unknown generate

# Update existing intent
unknown update

# Search recorded intent
unknown search

# Check repository configuration and metadata
unknown doctor
```

------------------------------------------------------------------------

## Privacy and Trust

Engineering context can contain sensitive information. Privacy must be a
core design requirement.

The product should:

-   Capture only inputs that the developer or organization has
    authorized.
-   Avoid collecting secrets, credentials, and unrelated personal data.
-   Make it clear when source code or context is sent to an external AI
    provider.
-   Support configurable provider and data-handling policies.
-   Keep generated intent reviewable and editable.
-   Distinguish observed facts, developer-authored statements, and
    AI-generated suggestions.
-   Preserve provenance so engineers can understand where a statement
    came from.
-   Avoid storing private model reasoning; store concise, reviewable
    engineering rationale instead.

Specific retention, encryption, access-control, and enterprise policy
behavior must be defined before production deployment.

------------------------------------------------------------------------

## Project Status

**Unknown is an early-stage product concept and architecture.**

The architecture, feature set, manifest example, technology choices,
CLI, and roadmap in this README describe the proposed direction. They
should not be interpreted as evidence that those features or commands
are already implemented.

Initial implementation priorities:

1.  Define and validate the `.unknown/` manifest schema.
2.  Build a minimal VS Code extension workflow.
3.  Generate and review intent for a selected code component.
4.  Persist approved intent in the repository.
5.  Add structural identity with clear handling for uncertain matches.

### Open design questions

-   What is the canonical identity model for a component across split
    and merge refactors?
-   Should manifests be organized per component, per file, or through a
    normalized index?
-   Which AI-assisted development environments can expose conversation
    context through supported APIs?
-   How should Unknown represent conflicting or superseded decisions?
-   Which operations must work entirely offline?
-   How should intent provenance and confidence be represented?

These questions should be resolved through small prototypes and explicit
design decisions rather than assumed in advance.

------------------------------------------------------------------------

## Contributing

Contributions, design discussions, and implementation experiments are
welcome as the project takes shape.

Before contributing:

1.  Review the project goals and current implementation status.
2.  Open an issue to discuss substantial architectural changes.
3.  Keep changes focused and include tests where applicable.
4.  Update documentation when behavior or design contracts change.
5.  Never include secrets, private prompts, or proprietary source
    material in examples.

Contribution guidelines, development setup, and code of conduct can be
added when the repository is ready to accept external contributions.

## License

A license has not yet been selected. Until one is added to the
repository, no open-source usage or redistribution permissions should be
assumed.
