# Project Decisions

This document is a historical record of significant product, architecture, taxonomy, governance, and engineering decisions made for Bug Museum.

It is not a changelog and does not track every implementation detail. It exists so that future maintainers and contributors can understand why the project looks the way it does, without having to reconstruct the reasoning from commit history.

Decisions are recorded in a lightweight Architecture Decision Record (ADR) style: Context, Decision, Reasoning, Consequences. Once accepted, a decision is not edited to reflect new thinking. If circumstances change, a new decision supersedes it and the old entry's status is updated accordingly.

---

## Decision 001 — Treat bug entries as structured content

**Status:** Accepted
**Date:** 2026-08-01

### Context
Bug entries could be written as free-form prose or as structured content with consistent fields.

### Decision
Bug entries use structured metadata/frontmatter.

### Reasoning
Structured metadata allows entries to eventually support validation, indexing, search, companion websites, datasets, and tooling, without requiring a rewrite of existing content later.

### Consequences
Every entry must carry its required metadata to remain valid. This adds a small amount of upfront overhead per entry in exchange for future extensibility.

---

## Decision 002 — Use folders for primary classification and metadata/tags for cross-cutting classification

**Status:** Accepted
**Date:** 2026-08-01

### Context
A bug can often be described by more than one characteristic (for example, the layer it occurs in versus the kind of failure it represents). The repository needed a rule for where an entry physically lives versus how it is otherwise described.

### Decision
A bug has one primary physical location (its folder), while secondary characteristics are represented through metadata/tags rather than duplicate folder placement.

### Reasoning
A single primary location keeps the repository structure simple and avoids ambiguity about where an entry lives. Tags provide the flexibility to express cross-cutting characteristics without multiplying folders.

### Consequences
Navigating by folder gives a primary classification; navigating by tag (once tooling exists) gives cross-cutting views. No entry should need to be duplicated across folders to reflect a secondary characteristic.

---

## Decision 003 — Start with only three bug categories

**Status:** Accepted
**Date:** 2026-08-01

### Context
A large taxonomy could be designed upfront, but the repository has limited content so far, and an early taxonomy risks being wrong.

### Decision
The initial categories are:

- authentication
- api
- frontend

### Reasoning
Starting narrow avoids empty or speculative categories that make the repository look larger or more organized than its actual content supports. New categories should be introduced when real content justifies them, not in anticipation of it.

### Consequences
Category growth is deliberate and content-driven. Contributors proposing a new category should expect to justify it with actual entries, not just a taxonomy argument.

---

## Decision 004 — Build Bug Museum as an engineering learning resource

**Status:** Accepted
**Date:** 2026-08-01

### Context
The project could function as a simple catalog of known defects, or as something that teaches engineering judgment.

### Decision
Entries teach root cause, investigation, QA detection, automation, prevention, lessons learned, and related engineering concepts, rather than merely cataloguing that a defect occurred.

### Reasoning
A catalog of defects has limited long-term value once the novelty wears off. A resource that teaches transferable judgment remains useful regardless of whether a reader has seen that specific bug before.

### Consequences
Entries that describe a bug accurately but teach no transferable lesson do not meet the bar for inclusion. This raises the effort required per entry but keeps the collection valuable at scale.

---

## Decision 005 — Use evidence-based sourcing

**Status:** Accepted
**Date:** 2026-08-01

### Context
Bug entries reference real-world incidents, which creates risk if claims about specific companies or individuals cannot be substantiated.

### Decision
Claims about real-world incidents must use reliable public sources or appropriately anonymized examples.

### Reasoning
Unverifiable claims about companies or individuals expose the project to credibility and reputational risk, and undermine the goal of being a resource engineers can trust and cite.

### Consequences
Contributors must be able to point to a public source or anonymize an example convincingly. Entries that cannot meet this bar should be reworked or declined rather than merged as-is.

---

## Decision 006 — Keep the repository simple before adding automation

**Status:** Accepted
**Date:** 2026-08-01

### Context
CI workflows, content generators, companion websites, and other tooling could be built early in anticipation of future scale.

### Decision
Do not introduce CI workflows, generators, websites, or other unnecessary tooling merely because they may be useful later.

### Reasoning
Tooling built ahead of actual content and usage tends to guess wrong about what is actually needed, and adds maintenance burden without proportional benefit while the repository is still small.

### Consequences
Tooling decisions are deferred until the repository has enough content or contributor volume to justify them. This trades some short-term convenience for avoiding premature, possibly wrong, investment.

---

## Decision 007 — Maintain a separate learning layer

**Status:** Accepted
**Date:** 2026-08-01

### Context
Some lessons span multiple bug entries rather than belonging to any single one, such as recurring engineering patterns or anti-patterns.

### Decision
A `learn/` area captures reusable engineering patterns, testing lessons, and anti-patterns discovered across multiple exhibits.

### Reasoning
Cross-cutting lessons lose value if they are buried inside a single bug entry or repeated across several. A dedicated area lets this knowledge be maintained once and referenced from multiple entries.

### Consequences
Entries that surface a generalizable lesson should link to or contribute toward the `learn/` area rather than restating the lesson in full each time it recurs.

---

## Decision 008 — Treat Bug Museum as a recognizable open-source product

**Status:** Accepted
**Date:** 2026-08-01

### Context
The project's museum/exhibit framing could be applied loosely, dropped entirely, or used inconsistently across the repository.

### Decision
Use the museum/exhibit concept consistently where useful, while maintaining professional engineering language and avoiding gimmicks.

### Reasoning
A consistent identity makes the project more recognizable and citable, but only if it does not come at the expense of the professional, engineering-focused tone the content depends on for credibility.

### Consequences
Naming and framing choices should reinforce the museum/exhibit identity without introducing cute or unprofessional language into technical content.
