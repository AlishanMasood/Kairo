# Documentation Standards

## Metadata

| Field | Value |
|-------|-------|
| Document ID | KAI-GOV-001 |
| Status | Draft |
| Version | 0.1 |
| Owner | Documentation Architect |
| Created | 2026-07-14 |
| Last Updated | 2026-07-14 |

---

## Purpose

Documentation standards ensure consistency, clarity, and long-term maintainability across all documents in the Kairo Architecture Repository. Without enforced standards, documentation degrades over time, becomes contradictory, and loses its value as a decision-making tool.

This document defines the rules that every document in this repository must follow.

## Scope

These standards apply to every document stored in this repository, regardless of folder, author, or subject matter. All contributors are expected to follow these conventions when creating or modifying documentation.

## Documentation Principles

- **Single Source of Truth** — Each concept, decision, or specification is documented in exactly one place. Other documents reference it rather than duplicating it.
- **Documentation Before Implementation** — Document intent and design before building. Code follows documentation, not the reverse.
- **Simplicity** — Write only what is necessary. Remove unnecessary complexity.
- **Consistency** — Follow the same structure, terminology, and formatting across all documents.
- **Version Awareness** — Documents must clearly indicate which version of the platform they apply to.
- **Traceability** — Every decision and specification should be traceable to its origin and related documents.
- **Incremental Documentation** — Documents evolve over time. Start with what is known and expand as clarity increases.
- **Long-Term Maintainability** — Write for the reader six months from now, not for today's meeting.

## Standard Document Metadata

Every document in this repository must begin with a metadata section in table format immediately after the document title.

Required fields:

| Field | Description |
|-------|-------------|
| Title | Full document title |
| Document ID | Unique identifier following the naming scheme (e.g., KAI-ARCH-001) |
| Status | Current lifecycle status (see Document Status section) |
| Version | Current version number |
| Target Release | The platform release this document targets, if applicable |
| Owner | The person or role responsible for maintaining this document |
| Created | Date the document was first created (YYYY-MM-DD) |
| Last Updated | Date of the most recent modification (YYYY-MM-DD) |
| Reviewers | List of individuals or roles who reviewed this document |
| Related Documents | Relative links to related documents in this repository |

## Document Status

Every document must carry one of the following statuses:

| Status | Meaning |
|--------|---------|
| Draft | Initial creation. Content is incomplete or unreviewed. May change significantly. |
| Review | Content is complete and submitted for peer review. Changes are expected based on feedback. |
| Approved | Reviewed and accepted. Represents the current agreed-upon state. |
| Implemented | The content described in the document has been realized in the platform. |
| Deprecated | No longer current. Retained for historical reference. A superseding document should be linked. |

Status transitions follow this order:

```
Draft -> Review -> Approved -> Implemented -> Deprecated
```

A document may return from Review to Draft if significant rework is needed.

## Versioning Rules

Documents follow semantic versioning adapted for documentation:

- **0.x** — Draft versions. Content is incomplete or under active development.
- **1.0** — First approved version.
- **1.x** — Minor revisions to approved content (corrections, clarifications, additions that do not change intent).
- **2.0+** — Major version increase for significant structural changes, scope redefinition, or content that supersedes previous meaning.

Rules:

- Increment the minor version for every approved change after initial approval.
- Increment the major version when the document's fundamental scope or structure changes.
- Always update the Last Updated field when changing the version.
- Never reuse a version number.

## Writing Style

All documents must adhere to the following writing conventions:

- Use professional, objective language.
- Write in third person or imperative mood.
- Use engineering terminology appropriate to the audience.
- Avoid marketing language, superlatives, and subjective claims.
- Avoid ambiguity. If a statement can be interpreted multiple ways, rewrite it.
- Prefer bulleted or numbered lists over long paragraphs.
- Keep sentences short and direct.
- Define acronyms on first use.
- Do not use emojis.
- Do not use badges or decorative elements.

## Markdown Standards

### Heading Hierarchy

- `#` — Document title (exactly one per document)
- `##` — Major sections
- `###` — Subsections
- `####` — Sub-subsections (use sparingly)

Do not skip heading levels.

### Tables

Use tables for structured data, metadata, and comparisons. Align columns for readability in source.

### Lists

- Use `-` for unordered lists.
- Use `1.` for ordered lists where sequence matters.
- Nest lists with consistent indentation (two or four spaces).

### Code Blocks

Use fenced code blocks with language identifiers:

````
```yaml
key: value
```
````

### Mermaid Diagrams

Use Mermaid fenced blocks for diagrams that benefit from version control:

````
```mermaid
graph TD
    A --> B
```
````

Keep diagrams simple. If a diagram requires more than 20 nodes, consider splitting it.

### Relative Links

Always use relative paths when linking to other documents in this repository:

```
[Architecture Overview](../04-Architecture/System-Architecture.md)
```

Never use absolute URLs for internal documents.

## File Naming Convention

All document files must follow these rules:

- Use PascalCase with hyphens separating words: `System-Architecture.md`
- Use `.md` extension for all documents.
- Names must be descriptive and specific.

Prohibited terms in file names:

| Term | Reason |
|------|--------|
| final | Implies no further changes; incompatible with versioning |
| new | Relative to a point in time; meaningless after that moment |
| latest | Ambiguous; the repository itself tracks what is latest |
| v2, v3 | Version belongs in metadata, not file names |
| copy | Indicates duplication rather than proper branching |
| draft | Status belongs in metadata, not file names |

## Folder Responsibility

| Folder | Responsibility |
|--------|---------------|
| 00-Governance | Standards, policies, and decision-making frameworks |
| 01-Foundation | Core principles, glossary, and foundational definitions |
| 02-Products | Product specifications and feature definitions |
| 03-Roadmap | Planning, milestones, and timeline documentation |
| 04-Architecture | System architecture and technical design |
| 05-Development | Development processes, guidelines, and tooling |
| 06-Modules | Per-module documentation and specifications |
| 07-Releases | Release notes, changelogs, and migration guides |
| 08-ADR | Architecture Decision Records |
| 09-Research | Explorations, spikes, and feasibility studies |
| 10-Templates | Reusable document templates |

Place documents in the folder that matches their primary purpose. Do not duplicate documents across folders.

## Cross-Referencing

Documents should reference related documents using relative Markdown links:

```markdown
See [Documentation Standards](../00-Governance/Documentation-Standards.md) for formatting rules.
```

Rules for cross-referencing:

- Always use relative paths from the current document's location.
- Link to the specific document, not the folder.
- If referencing a specific section, use anchor links: `[Section Name](./Document.md#section-name)`
- Keep the Related Documents metadata field updated with all documents that have a direct relationship.
- If a referenced document is deprecated, update the link to point to its replacement.

## Change Management

All documentation changes follow this process:

1. **Discussion** — Identify the need for a new document or a change to an existing one. Discuss scope and ownership.
2. **Draft** — Author creates or modifies the document. Status is set to Draft.
3. **Review** — Document is submitted for review. Reviewers are listed in metadata. Feedback is incorporated.
4. **Approval** — Designated approver accepts the document. Status moves to Approved. Version is incremented.
5. **Implementation** — When the documented content is realized in the platform, status moves to Implemented.

Rules:

- All changes to Approved documents require a review cycle.
- Trivial fixes (typos, broken links) may bypass full review but must still increment the minor version.
- Deprecated documents must link to their replacement.

## Future Compatibility

Documentation should describe current and planned states without forcing implementation of future features. When referencing future capabilities:

- Clearly mark forward-looking statements with the target release version.
- Do not write documentation that assumes unbuilt features exist today.
- Structure documents so that future sections can be added without restructuring existing content.
- Prefer additive changes over modifications to existing approved content.
