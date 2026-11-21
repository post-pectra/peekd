# Documentation guide

## Writing style

- Write only what the task needs, sized to its complexity.
- Use clear, complete sentences.
- Keep sentences short, with one idea each.
- Define an unfamiliar term the first time it appears.
- Leave out filler, repetition, excessive detail, and side topics nobody asked for.
- Do not repeat a fact that was already stated just to emphasize it.
- Do not use em dashes or inflated introductions.

## README.md

The README is the entry document for someone seeing the repository for the first time.

It includes:

- Purpose and scope of the project
- Main components
- Prerequisites
- How to run and test
- Link to the code wiki

It does not include:

- Rules that only apply to AI agents
- Detailed architecture and decision history
- Work plans and progress
- Explanations that are obvious from the code
- Copies of other documents

## Code wiki

The repository maintains these documents. `docs/README.md` is the map.

- `docs/architecture.md`: components, responsibilities, call and data flow
- `docs/glossary.md`: domain terms, abbreviations, state values
- `docs/adr/`: important technical decisions and the reasons for them

The README links to the code wiki and does not copy its content.

When a code change alters the architecture or a domain definition, update the related document in the same change.

## ADR

Write an ADR when any of the following applies.

- One option was chosen among several alternatives
- The decision affects several services or teams
- An API, event, or DB contract changes
- The decision affects security or assets
- The decision is hard to reverse or adds operational burden
- The reason for the choice is likely to be looked up again later

An ADR states only the problem, alternatives, decision, reasons, and impact, briefly.

File naming and numbering are in `docs/adr/README.md`.

## Commit messages

Commit messages are in English and use this form.

```text
<prefix>: <summary>

<reason for the change>
```

- The prefix is one of the two forms found in `git log`:
  - the area being changed, named after the package or directory: `processor: ...`, `p2p: ...`
  - a Conventional Commits type with an optional scope: `feat(processor): ...`, `fix(migrations): ...`, `chore: ...`, `docs: ...`, `refactor(processor): ...`
- The summary starts with a lowercase verb in the imperative mood: `add`, `extract`, `reduce`.
- The body explains why the change was made and any trade-off. A change whose summary is self-explanatory may omit it.
