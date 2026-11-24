---
name: project-docs
description: Concise, structured Markdown documentation for features and classes in software projects. Use when asked to write, update or review feature docs, class docs, READMEs or docs/*.md files.
---

# Documentation Style

Docs are read by humans first: short, scannable, accurate. Existing project doc conventions win. Never document what the code already says obviously.

## Principles
- Lead with purpose: what it is and why it exists, in 1-2 sentences.
- One topic per file. Link to related docs instead of repeating them.
- Bullets, tables and short sentences over paragraphs. No filler, no marketing words, no restating method names.
- Document the non-obvious: rules, constraints, side effects, error cases, decisions and their reason.
- Facts only from the code. Verify names, endpoints, fields and error codes before writing; mark unknowns `TODO` instead of guessing.
- Update the existing doc rather than creating a duplicate. Keep it in sync when behavior changes.
- No secrets, tokens or real data in examples.

## Formatting
- One `#` title; `##` sections; no deeper than `###`.
- Fixed section order (below); drop sections that do not apply, never pad.
- Code, class, method, endpoint and field names in backticks. Code blocks tagged with a language.
- Tables for fields, endpoints, error codes. Max ~5 columns, one line per cell.
- Target: feature doc <= 1 page, class doc <= half a page. Past that, split.

## Feature doc (`docs/features/<feature>.md`)
```
# <Feature name>
<Purpose: what it does and for whom, 1-2 sentences.>

## Flow
1. <Step, naming the class/endpoint involved>
2. ...

## Rules
- <Business rule / validation / permission>

## API            (only if it exposes endpoints)
| Method | Path | Purpose | Errors |

## Data           (only if it owns tables/entities)
| Entity/Table | Key fields | Notes |

## Errors
| Code | When |

## Notes
- <Edge cases, side effects (events, jobs, emails), decisions and why>

## Related
- <links to code or other docs>
```

## Class doc (`docs/classes/<Class>.md`, or Javadoc for small classes)
```
# <ClassName>
<Responsibility: one sentence. Layer/type: service, mapper, guard...>

## Dependencies
- `OtherService` - why it is used

## Public methods
| Method | Does | Throws |
| `name(args)` | <one line> | `ErrorCode.X` when ... |

## Notes
- <Transaction behavior, ordering, caching, thread-safety, gotchas>
```
Skip trivial classes (plain DTOs, simple mappers): one line in the feature doc is enough.

## Writing checklist
- Can a new teammate grasp the purpose in 10 seconds?
- Every line earns its place; nothing duplicates the code or another doc.
- Names, paths and codes match the code now.
- Tables render and headings follow the fixed order.
