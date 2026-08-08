# Bug Museum Roadmap

This describes direction, not a schedule. There are no dates or deadlines here; priorities will shift as the project learns from actual exhibits and actual contributors, not from a plan written in advance of either.

## Current Foundation

What exists today:

- Repository architecture: folder-per-category structure with metadata/tags for cross-cutting classification.
- A public record of project decisions ([PROJECT_DECISIONS.md](PROJECT_DECISIONS.md)).
- A canonical exhibit template ([templates/BUG_ENTRY_TEMPLATE.md](templates/BUG_ENTRY_TEMPLATE.md)).
- Contribution guidelines ([CONTRIBUTING.md](CONTRIBUTING.md)).
- One published exhibit, in the Authentication category.

## Now

The immediate objective is to build a small collection of exceptionally strong exhibits across the three approved categories: Authentication, API, and Frontend. Quality matters more than reaching any particular exhibit count.

Alongside that:

- Refining the exhibit format based on what real entries actually need, not on speculation.
- Improving cross-linking between exhibits as related bugs appear.
- Learning which metadata fields are genuinely useful once there is enough content to tell.

## Next

Once there are enough exhibits to make these worth doing:

- Additional high-quality exhibits in the existing categories.
- Stronger navigation within and across categories.
- Reusable engineering lessons extracted into `learn/`, once patterns repeat across more than one exhibit.
- Contributor-experience improvements driven by what actual contributors run into, not anticipated friction.
- Basic validation, only if and when maintaining consistency by hand becomes genuinely difficult.

None of this implies a specific tool or technology choice yet; that gets decided when the need is concrete.

## Later

These are possibilities worth naming, not commitments:

- A searchable static website over the exhibit collection.
- A generated exhibit index.
- Filtering by category, difficulty, production impact, or failure pattern.
- Richer diagrams for exhibits describing complex, multi-step failures.
- A structured dataset or export of exhibit metadata.
- Educational "spot the bug" style exercises built from existing exhibits.
- Additional categories, once real content justifies them.

## What We Are Deliberately Not Doing Yet

Complex CI pipelines, custom tooling, an elaborate taxonomy, a long list of empty categories, website infrastructure, and contributor process for its own sake are all things this project is intentionally not building right now.

The principle is simple: complexity should be earned by an actual, demonstrated need, not installed in advance of one. A structure built before there is content to justify it usually guesses wrong about what that content will need.

## How the Roadmap Evolves

This roadmap will change as the project learns. What actually happens while writing exhibits, feedback from contributors, failure patterns that keep recurring across entries, the maintenance cost of any structure that gets added, and whether something turns out to be genuinely useful to engineers, all outweigh whatever was assumed here in advance.
