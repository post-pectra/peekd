# Architecture Decision Records

An ADR records an important technical decision and why it was chosen.

## When to write one

Write an ADR when any of the following applies.

- One option was chosen among several alternatives
- The decision affects several services or teams
- An API, event, or DB contract changes
- The decision affects security or assets
- The decision is hard to reverse or adds operational burden
- The reason for the choice is likely to be looked up again later

## File names

- `ADR-<number>-<decision-name>.md`, for example `ADR-0001-decision-name.md`
- The number has four digits and increases by one from the latest ADR in this directory.
- The decision name is lowercase English words joined with hyphens.

## Content

State the problem, alternatives, decision, reasons, and impact, briefly.

## Changing a decision

Do not rewrite an existing ADR with the new decision. Write a new ADR and mark the old one as superseded by the new number.
