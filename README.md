# Bug Museum

A curated collection of real, recurring software bugs, documented for the engineering judgment they teach, not just the failure they represent.

## What Is Bug Museum

Bug Museum is an open-source repository that catalogs software bugs that recur across companies, stacks, and years. Each entry is called an exhibit: a structured writeup of a real failure that goes past "here's what broke" into why it broke, how it could have been caught, and how to prevent the same class of failure next time.

This is not an incident tracker, and it is not a running list of "interesting bugs someone found." Every exhibit exists to teach something transferable. If a bug can only be understood by someone who worked at the company where it happened, it does not belong here.

The goal is for entries to hold up on their own years from now, the way a well-written postmortem does, regardless of whether the reader has ever seen that specific bug before.

## Why Bug Museum Exists

Most engineering teams encounter the same handful of failure categories, over and over, in slightly different clothing. An off-by-one in pagination. A race condition in a token refresh. A frontend state bug that only appears after a specific sequence of navigation. The bug gets fixed, the incident channel goes quiet, and the lesson usually leaves the company with the engineer who learned it.

Bug Museum exists to keep that lesson around. Instead of letting engineering judgment evaporate after an incident is resolved, this project documents the failure, the reasoning behind it, and how to detect and prevent its recurrence, in a form that outlives the individual who wrote it up.

## What an Exhibit Teaches

An exhibit is not a bug report. It is a structured breakdown of a failure from multiple engineering perspectives:

- **What happened** — symptoms, root cause, and why the failure occurs in the first place
- **Who it affects** — the actual user impact, not just the technical defect
- **How to find it** — the investigation path a developer would follow, and the strategy a QA engineer would use to catch it before release
- **How to prevent it** — an automation strategy for catching the same class of bug going forward, plus the broader lesson learned
- **Where else it shows up** — related bugs and credible references for further reading

Every exhibit includes structured metadata (difficulty, production impact) so entries stay comparable and, eventually, searchable.

## Explore the Museum

The museum is organized into wings, one per bug category. Right now, there are three:

- [Authentication](bugs/authentication)
- [API](bugs/api)
- [Frontend](bugs/frontend)

New wings are added when there is enough meaningful content to justify one, not in anticipation of content that doesn't exist yet.

## Who This Is For

- **QA Engineers** looking for real failure patterns to inform test design
- **SDETs / Test Automation Engineers** looking for cases worth automating and why
- **Software Engineers** who want to recognize a failure class before they ship it
- **Engineering Managers / Tech Leads** looking for material to sharpen a team's debugging instincts
- **Students and early-career engineers** building an intuition for how real systems actually fail

## Engineering Philosophy

- **Quality over quantity.** A small number of excellent exhibits is worth more than a large number of shallow ones.
- **Evidence-based sourcing.** Claims about real-world incidents rely on credible public sources or clearly anonymized examples, not speculation.
- **Learn from failure, don't just log it.** An exhibit that doesn't teach something transferable isn't finished.
- **Prevention over cataloguing.** The point of documenting a bug is to make its class of failure less likely to happen again.
- **Simple architecture until complexity is earned.** Structure, tooling, and process are added when the project's actual scale justifies them, not ahead of it.

## Contributing

Contributions are welcome, and the bar is quality, not volume. A single well-documented exhibit is worth more than several thin ones. See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose one.

## Project Decisions

Significant architecture, taxonomy, and governance decisions are documented publicly in [PROJECT_DECISIONS.md](PROJECT_DECISIONS.md), so future contributors can understand why the project is shaped the way it is without having to reconstruct the reasoning from commit history.

## Roadmap

Planned direction for the project is tracked in [ROADMAP.md](ROADMAP.md).

## License

The educational and documentation content in this repository is available under the [Creative Commons Attribution 4.0 International license](LICENSE) (CC BY 4.0).
