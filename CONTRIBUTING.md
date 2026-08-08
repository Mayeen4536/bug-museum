# Contributing to Bug Museum

Bug Museum is early, with a single exhibit so far, so this process is intentionally lightweight. It will get more structure as the repository grows, but for now the bar is judgment, not paperwork.

## What Makes a Good Contribution

A good exhibit:

- teaches a transferable engineering lesson, not just a record of what broke
- explains more than the symptom
- distinguishes evidence from assumption
- avoids unverifiable claims about companies or individuals
- is understandable by someone outside the original system where the bug appeared
- adds meaningful value rather than simply increasing the entry count

## Ways to Contribute

Contributions are not limited to new exhibits. All of the following are welcome:

- new exhibits
- corrections to existing exhibits
- stronger or more precise references
- clearer explanations of an existing section
- links between related bugs
- suggestions about taxonomy or project structure

## Before Writing an Exhibit

1. Search existing exhibits so the same bug isn't documented twice.
2. Choose the most appropriate existing category. See [README.md](README.md) for the current list.
3. Start from [templates/BUG_ENTRY_TEMPLATE.md](templates/BUG_ENTRY_TEMPLATE.md).
4. Gather credible references for any factual claim you plan to make.
5. Anonymize anything private or company-sensitive before it goes into the exhibit.

## Creating an Exhibit

- Copy the approved template rather than writing structure from scratch.
- Use a descriptive, kebab-case filename that matches the `slug` field.
- Keep the YAML frontmatter and fill in every field that applies.
- Complete every section that is relevant to the bug; don't leave a section as a stub.
- Remove the template's instructional HTML comments once you've written real content in their place.
- If a section genuinely doesn't apply, it's fine to leave it out rather than force filler text into it.

## Evidence and Attribution

Bug Museum entries make factual claims, so sourcing matters. Acceptable evidence includes:

- standards documents and RFCs
- official vendor documentation
- public postmortems
- issue trackers or official repositories
- reputable engineering publications
- clearly anonymized first-hand patterns, when no identifiable person or company is being named

The following will be rejected:

- rumors
- unverifiable accusations
- invented incidents
- confidential or internal company material
- copied proprietary content

## Writing Expectations

- Clear engineering language over cleverness.
- No unnecessary hype.
- Technology-neutral reasoning where the lesson allows it.
- An explicit distinction between what was observed and what was inferred as the root cause.
- No fabricated statistics.
- No more certainty than the evidence supports.

## Pull Request Expectations

The workflow is basic:

1. Fork the repository.
2. Create a focused branch for one change.
3. Make the change.
4. Self-review it before opening a pull request.
5. Open a pull request.

In the pull request description, explain what changed, why it's useful, and which sources you relied on if the change involves factual claims.

## Scope and New Categories

New top-level bug categories aren't added casually. If an exhibit doesn't fit any existing category, explain the proposed category and the reasoning behind it before introducing the folder, rather than creating it directly. Raise this in the pull request itself, or through whatever discussion or issue mechanism the repository has available once those workflows are established.
