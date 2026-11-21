# Coding guide

## Implementation

- Follow the existing architecture and naming conventions. The component layout is in `docs/architecture.md`.
- Do not bundle refactoring that is unrelated to the request.
- Before adding a new dependency or pattern, check whether the existing approach can solve the problem.
- When fixing a bug, add a test that reproduces it where possible.
- When changing an API, event, or DB schema, check backward compatibility. The Go insert code in `repository/` and the table definitions in `migrations/` must stay in step.
- For external system integration, consider timeout, retry, idempotency, and partial failure.
- Write code comments in English.

## Verification

The module targets Go 1.24.1 (`go.mod`), and the `Dockerfile` builds with `CGO_ENABLED=1`, so a C toolchain is needed. Run all of the following from the repository root before calling a change done.

```bash
gofmt -l .                 # must print nothing
go vet ./...
go test ./...
go build -o peekd ./cmd
```

- There is no Makefile and CI does not run tests. The workflow only runs a SonarCloud scan and the Docker image build, so local verification is the only check.
- When the `Dockerfile` or dependencies change, also run `docker build .`.
- To run the tool with its database locally, follow the Quick Start in `README.md`.

## Branches

- Branch from `dev` and open pull requests against `dev`.

## Documentation impact

For the following changes, check whether the code wiki needs an update, following `.agents/documentation.md`.

- A change to the main components or their responsibilities
- A change to the call or data flow
- A new domain term or state value
- An important technical decision
