# peekd code wiki

Technical documents that live with the code and change with it. Start from the [README](../README.md) for what peekd is and how to run it.

## Map

| Document | Content |
| --- | --- |
| [architecture.md](architecture.md) | Purpose and scope, components and their responsibilities, data flow, external systems, data ownership, operational boundaries |
| [glossary.md](glossary.md) | Domain terms, abbreviations, and column meanings used in the code and the database |
| [adr/](adr/README.md) | Important technical decisions and the reasons for them |

Related material outside `docs/`:

| Location | Content |
| --- | --- |
| [README.md](../README.md) | Overview diagram, key design decisions, supported message types, quick start, configuration |
| [migrations/](../migrations) | ClickHouse table and materialized view definitions, with a comment per column |
| [.env.example](../.env.example) | Environment variables read at startup |

## Writing criteria

- Write what was confirmed in the code. Do not write guesses.
- Do not list files, types, or functions that are visible directly in the code.
- Do not copy content that another document already has. Link to it.
- Update the document in the same change that alters the architecture, a term, or a column meaning.
- Record a decision in `adr/` when it meets the conditions in [adr/README.md](adr/README.md).
