---
name: Knowledge Base Architect
description: Guidelines and tools for building a well-connected web development knowledge base in Obsidian.
---

# Knowledge Base Architect Skill

This skill provides a systematic approach to creating and organizing technical notes within your Obsidian vault. It emphasizes a **flat folder structure**, **Map of Content (MOC)** indexing, and **atomic note-taking**.

## Core Principles

1.  **Atomic Notes**: Each note should focus on a single concept (e.g., "Closure" rather than "Advanced JavaScript").
2.  **Flat Folder Structure**: All notes reside in the `Notes/` directory. Organization is achieved through tags and internal links.
3.  **Map of Content (MOC)**: Top-level notes that act as hubs for a specific topic (e.g., `Frontend MOC.md`).
4.  **Sentence Case**: Note titles must be in sentence case (e.g., `React server components.md`).
5.  **Linking Over Categorization**: Always look for opportunities to link a new note to existing ones using `[[Note Name]]`.

## Note Types & Tags

| Note Type | Tag | Purpose |
| :--- | :--- | :--- |
| **Concept** | `#concept` | Fundamentals, definitions, and mental models. |
| **How-to** | `#how-to` | Practical guides, configurations, code snippets. |
| **Resource** | `#resource` | Curated lists of external links, docs, or books. |
| **MOC** | `#moc` | Higher-level index for a topic area. |

## Metadata (YAML Frontmatter)

Every note should start with the following frontmatter:

```yaml
---
created: YYYY-MM-DD
tags: [concept]
related: [ [[Parent Note]], [[Related Note]] ]
status: draft
---
```

## Usage for the AI

When tasked with "Adding a note" or "Improving the KB":

1.  **Check for Duplicates**: Search for existing notes on the topic.
2.  **Select Template**: Use the appropriate template from `Agent Skills/knowledge-base-architect/templates/`.
3.  **Draft Content**: Keep it concise. Use code blocks for snippets.
4.  **Connect**:
    - Add at least 2 internal links to other relevant notes.
    - Check the relevant **MOC** and add the new note to its index.
5.  **Tag**: Ensure the correct functional tag is applied.
