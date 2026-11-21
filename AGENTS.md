# AGENTS.md

peekd is a standalone Ethereum P2P network diagnostic tool. It joins the beacon chain P2P network, collects gossip messages, and stores per-message metrics in ClickHouse.

## Language

- Write documents, code comments, commit messages, and work results in English.
- Keep code identifiers, API and protocol names, file names, and commands as they are spelled in the code.

## Task guides

- For code work, read and follow `.agents/coding.md`.
- For documentation work such as the README, the code wiki, and commits, read and follow `.agents/documentation.md`.
- When a change touches both code and documents, read both guides.
- Do not read guides that are unrelated to the task.

## Team safety rules

Do not run the following without approval from the person in charge.

- Production deploy, restart, or data change
- Creating, reading, or rotating keys and secrets
- Changing or deleting infrastructure resources or security policies

## Rules for this repository

- The default branch is `dev`. A push to `dev` or a `v*` tag runs `.github/workflows/dev.yaml`, which builds the Docker image and pushes it to Docker Hub with the `latest` tag. Treat both as a production deploy.
- Runtime settings, including the node key (`PEEKD_ECDSA_PRIVATE_KEY_HEX`) and the database password (`PEEKD_DB_PASSWORD`), come from `.env`. Do not commit `.env` or put real values in `.env.example`.
