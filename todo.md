# tpt-praxis — Project Todo

Source: [spec.txt](spec.txt). Scope: HTTP + MCP for v1, gRPC deferred. Dependencies are TPT crates on GitHub (MIT OR Apache-2.0). License: MIT OR Apache-2.0, copyright TPT Solutions. Dependency rule: allow-list only (MIT, or dual MIT OR Apache-2.0); Apache-only crates rejected.

## Phase 0 — Decisions & Prerequisites

- [x] GitHub owner/org: `tpt-solutions` (https://github.com/tpt-solutions)
- [ ] Confirm GitHub URLs for each TPT dependency under `tpt-solutions`: tpt-uir, tpt-norma, tpt-writ, tpt-concord, tpt-workflow, tpt-local-ai, tpt-primitives, tpt-crypto, tpt-assurance-harness
- [x] tpt-workflow and tpt-local-ai are in v1 (not deferred)
- [ ] Confirm each TPT dependency's license (MIT OR Apache-2.0) and that it passes the allow-list
- [ ] Confirm the MCP SDK (official Rust SDK, `rmcp`) license passes the allow-list; fall back to hand-rolled JSON-RPC if not
- [ ] Review tpt-concord's append/transaction API to decide how audit atomicity is achieved (see Phase 9)
- [ ] Confirm Rust toolchain version, edition, and MSRV
- [ ] Confirm the copyright line format (e.g. "Copyright (c) 2026 TPT Solutions")

## Phase 1 — Repository & Licensing

- [x] Initialise git in `c:\Programming\2 WIP\tpt-praxis`
- [ ] Add `LICENSE-MIT` and `LICENSE-APACHE` (standard texts)
- [ ] Add `README.md` with dual-license statement (`MIT OR Apache-2.0`) and TPT Solutions copyright
- [x] Add `.gitignore` for Rust (`target/`, editor files, `.env`)
- [ ] Create GitHub remote and push initial commit
- [ ] Add `CONTRIBUTING.md` (contributions licensed under MIT OR Apache-2.0)
- [ ] Add `SECURITY.md` (vulnerability reporting)

## Phase 2 — CI & Supply-Chain Guardrails

- [ ] Add GitHub Actions workflow: `cargo fmt --check`, `cargo clippy -D warnings`, `cargo test`, `cargo build --release`
- [ ] Add `deny.toml` for `cargo-deny`
  - [ ] Allow-list: MIT, Apache-2.0, dual `MIT OR Apache-2.0`, and any other license explicitly approved
  - [ ] Ban Apache-2.0-only dependencies
  - [ ] Ban copyleft (GPL, AGPL, LGPL)
  - [ ] Fail CI on advisories (RustSec)
- [ ] Add `cargo-deny` job to CI
- [ ] Add `cargo-audit` or equivalent advisory check to CI
- [ ] Add license-header check (if desired)

## Phase 3 — Cargo Workspace Scaffolding

- [ ] Create root `Cargo.toml` workspace with members:
  - [ ] `tpt-praxis-core`
  - [ ] `tpt-praxis-api`
  - [ ] `tpt-praxis-adapters`
  - [ ] `tpt-praxis-cli`
- [ ] Set shared `[workspace.package]` fields: license `MIT OR Apache-2.0`, authors "TPT Solutions", edition, MSRV
- [ ] Create empty crate skeletons with `lib.rs` / `main.rs` and a passing `cargo build`
- [ ] Set shared `[workspace.dependencies]` for TPT crates and third-party crates

## Phase 4 — TPT Ecosystem Dependency Verification

- [ ] Add git dependency on tpt-primitives and tpt-crypto (shared types, hashing)
- [ ] Add git dependency on tpt-uir (CanonicalAction envelope)
- [ ] Add git dependency on tpt-norma (policy evaluation; `DecisionEvidence`)
- [ ] Add git dependency on tpt-writ (permit minting and execution boundary)
- [ ] Add git dependency on tpt-concord (audit ledger)
- [ ] Add git dependency on tpt-workflow (v1, multi-step state transitions)
- [ ] Add git dependency on tpt-local-ai (v1, constrained decoding for raw LLM stream input)
- [ ] Add git dependency on tpt-assurance-harness (dev-dependency, for Phase 13)
- [ ] Pin each git dependency to a tag or commit SHA
- [ ] Confirm `cargo deny check` passes with all TPT git dependencies

## Phase 5 — tpt-praxis-core (Orchestration Loop)

- [ ] Define typed error enum covering: policy deny, policy indeterminate, state fetch failure, permit failure, state drift, execution failure, audit failure
- [ ] Define dependency-injection traits:
  - [ ] `StateStore` (fetch `StateSnapshot` + `StateVersion`)
  - [ ] `PolicyEvaluator` (wraps tpt-norma)
  - [ ] `PermitIssuer` / `Executor` (wraps tpt-writ)
  - [ ] `AuditSink` (wraps tpt-concord)
- [ ] Implement ingest & validate: parse request into `CanonicalAction` (tpt-uir)
- [ ] Implement fetch state step
- [ ] Implement policy evaluation step
  - [ ] Deny / Indeterminate: halt, log to audit, return error
  - [ ] Allow: receive `DecisionEvidence`
- [ ] Implement permit minting: pass `ActionDigest`, `StateVersion`, `DecisionEvidence` to tpt-writ
- [ ] Implement execute step: pass `ExecutionPermit` + `ActionRequest` to tpt-writ (never call the adapter directly)
- [ ] Implement audit step: append full lifecycle record regardless of outcome
- [ ] Implement `execute_protected_action` as the single entry point
- [ ] Enforce fail-closed: DB unreachable, audit full, policy timeout → error, never allow
- [ ] Add policy evaluation timeout
- [ ] Unit tests for each step and each fail-closed branch

## Phase 6 — In-Memory State Adapter

- [ ] Implement in-memory `StateStore` in `tpt-praxis-adapters`
- [ ] Implement versioned state (`StateVersion` increments on write)
- [ ] Implement in-memory side-effect driver for testing
- [ ] Implement in-memory audit sink (test-only) if tpt-concord needs a stand-in
- [ ] Tests for version increment and drift detection

## Phase 7 — tpt-praxis-api (HTTP via axum)

- [ ] Define request/response schema (JSON), with schema validation
- [ ] Implement `POST` endpoint for `execute_protected_action`
- [ ] Map errors to HTTP status codes (deny → 403, drift → 409, infra failure → 503, audit failure → 500)
- [ ] Return `200 OK` with a tpt-concord record reference on success
- [ ] Add authentication (decide scheme: API key or mTLS; confirm with user before Phase 7)
- [ ] Add request-size limits and timeouts
- [ ] Add routing and health endpoint
- [ ] Accept raw LLM stream input and wire tpt-local-ai constrained decoding

## Phase 8 — MCP Integration

- [ ] Add MCP server (official Rust SDK, if its license passes; otherwise hand-rolled)
- [ ] Expose one MCP tool that routes to `execute_protected_action`
- [ ] Generate tool JSON schema from the same typed request as HTTP
- [ ] Ensure MCP calls produce an audit record that can be traced back to the agent
- [ ] Test an MCP client calling the tool end-to-end

## Phase 9 — Audit Atomicity

- [ ] Design the atomicity mechanism (in-transaction append, two-phase, or compensating rollback) based on tpt-concord's API
- [ ] Implement: side effect and audit append treated as one logical transaction
- [ ] Implement rollback of the side effect on audit append failure
- [ ] If rollback is not possible, implement "flag for human intervention" path and alert
- [ ] Tests for rollback and flag paths

## Phase 10 — tpt-praxis-adapters (Real Backends)

- [ ] Decide first real backend: Postgres or SQLite (confirm before starting)
- [ ] Implement `StateStore` for chosen database
- [ ] Implement side-effect drivers (example: send email, actuate hardware) as separate modules behind traits
- [ ] Integration tests against a real database (docker-compose or testcontainers)
- [ ] Document how a domain adapter (e.g. tpt-healthcare-nz) is written

## Phase 11 — tpt-praxis-cli

- [ ] Subcommand: run the server locally
- [ ] Subcommand: inspect the tpt-concord audit ledger
- [ ] Subcommand: hot-reload tpt-norma policies
- [ ] Subcommand: run adversarial tests via tpt-assurance-harness
- [ ] CLI help text and usage examples

## Phase 12 — Acceptance Tests (Definition of Done)

- [ ] Positive: valid constrained JSON request → policy → execution → audit; returns 200 OK and a verifiable tpt-concord record
- [ ] Negative (policy denial): request violating tpt-norma constraint is blocked before the adapter; audit records Deny
- [ ] Negative (state drift): approved request whose state changes before execution is rejected by tpt-writ; audit records rejection
- [ ] Negative (audit failure): simulate full disk or locked DB for audit ledger; endpoint rejects action and rolls back side effect
- [ ] Integration (MCP): AI agent calls a tool through tpt-praxis-api; execution is traceable in the audit ledger
- [ ] All five acceptance tests run in CI

## Phase 13 — Adversarial & Fuzz Testing

- [ ] Fuzz `tpt-praxis-api` endpoints via tpt-assurance-harness
- [ ] Verify malformed requests never crash the orchestration loop
- [ ] Add fuzz/adversarial job to CI (nightly or on-demand)

## Phase 14 — Documentation

- [ ] Architecture doc: orchestration flow and invariants (from spec §4–5)
- [ ] API reference: HTTP and MCP endpoints
- [ ] Operator guide: audit ledger inspection, policy hot-reload
- [ ] Domain adapter guide: writing a new Postgres adapter and a new tpt-norma policy file
- [ ] Contributor guide and dependency-license policy

## Phase 15 — Release v0.1.0

- [ ] Verify all acceptance tests pass
- [ ] Verify `cargo deny` passes
- [ ] Write CHANGELOG.md
- [ ] Tag `v0.1.0` and publish release notes
- [ ] Decide whether to publish to crates.io (depends on whether TPT git dependencies are published)

## Phase 16 — Post-v1 (Deferred)

- [ ] gRPC API via tonic
- [ ] tpt-healthcare-nz domain adapter and policy
- [ ] tpt-payments domain adapter and policy
- [ ] tpt-flight-control domain adapter and policy
- [ ] Auditor / regulator query API (read-only views over tpt-concord)
