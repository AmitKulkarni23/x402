# Plan: Add Java Examples to x402

This is a working plan to mirror Python and TypeScript examples with Java examples using the existing `java/` SDK in this repo. You can reference this doc locally and avoid including it in PRs.

## What Was Reviewed
- Python examples: `examples/python/legacy/*` organized by category (clients, servers, fullstack, discovery) with per-example `pyproject.toml` and READMEs. Top-level `examples/python/legacy/README.md` explains structure, setup, and private key notes.
- TypeScript examples: `examples/typescript/*` similarly organized with a top-level `README.md`, `pnpm-workspace.yaml`, and per-example `package.json` + README.
- Java SDK present at `java/` with `pom.xml`, HTTP client (`X402HttpClient`), facilitator client interfaces, and a servlet `PaymentFilter` plus tests.

## Goals for Java Examples
- Provide runnable, minimal examples that mirror Python/TS categories.
- Keep per-example projects isolated with Maven, JDK 17, and clear README instructions.
- Reuse the existing `java/` SDK (install locally first), avoid changing core SDK unless required.

## Proposed Directory Structure
```
examples/
  java/
    README.md                   # overview, setup, category map, private key note
    clients/
      httpclient/               # plain Java 17 HttpClient + X402HttpClient
        pom.xml
        src/main/java/.../Main.java
        .env-local              # sample env vars
    servers/
      jetty-servlet/            # embedded Jetty + PaymentFilter registration
        pom.xml
        src/main/java/.../Main.java
        .env-local
      spring-boot-filter/       # Spring Boot app with PaymentFilter bean
        pom.xml
        src/main/java/.../Application.java
        src/main/java/.../FilterConfig.java
        .env-local
    discovery/
      http-facilitator/         # list supported kinds via HttpFacilitatorClient
        pom.xml
        src/main/java/.../Main.java
        .env-local
```
Optional follow-ups:
- `clients/okhttp/` (manual header construction with OkHttp/Retrofit)
- `fullstack/basic/` (Spring Boot app with tiny UI + protected route)
- Aggregator `examples/java/pom.xml` to build all examples at once

## Environment and Setup
- Prereqs: JDK 17, Maven.
- Build/install core Java SDK locally so examples can depend on it:
  ```bash
  mvn -q -f java/pom.xml -DskipTests install
  ```
- Each example:
  - Copy `.env-local` to `.env` (or export env vars directly)
  - Fill required vars: `X402_PRIVATE_KEY`, `X402_FACILITATOR_URL`, `X402_PAY_TO`, `X402_ASSET` (e.g., USDC)
  - Run via `mvn exec:java` (apps) or `mvn spring-boot:run` (Spring Boot)

## Signing Guidance
- Use a development key only. Never commit or use a mainnet-funded key.
- For EVM exact scheme (ERC-3009): provide a `CryptoSigner` implementation (e.g., using web3j) inside the example. Keep it scoped to examples and avoid exporting as SDK API unless needed.

## Example Content Outline
- `clients/httpclient`:
  - Minimal `Main.java` demonstrating building an `X-PAYMENT` header via `X402HttpClient.get(...)` and hitting a sample endpoint.
  - Simple `CryptoSigner` impl for EVM exact.
  - README covering env vars, run command, and expected output.
- `servers/jetty-servlet`:
  - Embedded Jetty server registering `PaymentFilter` with a price table (map path → amount in atomic units).
  - A basic servlet at `/private` returning JSON when paid.
  - README with run instructions and a curl snippet.
- `servers/spring-boot-filter`:
  - Spring Boot application with a `PaymentFilter` bean, protecting `/api/*`.
  - A controller returning JSON for a paid endpoint.
  - README with `mvn spring-boot:run` and test curl.
- `discovery/http-facilitator`:
  - Small console app calling facilitator `/supported` and printing kinds.
  - README with example output.

## Branching and PR Workflow
- Create a feature branch in your fork: `feat/java-examples`.
- Prefer small, reviewable PRs rather than a single large PR.
  - PR 1: `examples/java/README.md`, `clients/httpclient/`, `servers/jetty-servlet/`.
  - PR 2: `servers/spring-boot-filter/`, `discovery/http-facilitator/`.
  - PR 3+: Optional extras, aggregator POM.
- Open PRs against your fork’s default branch, then make an upstream PR (or open a draft upstream PR early for feedback).

## Tooling and Conventions
- Build: Maven per-example with JDK 17, depend on `com.coinbase:x402:1.0.0-SNAPSHOT` (from local install).
- Run: `mvn exec:java` or `mvn spring-boot:run`.
- Lint/format: keep example pom.xml minimal; avoid repo-wide config changes.
- Tests: optional smoke tests per example (don’t overbuild for initial PRs).

## Documentation Consistency
- Mirror tone/sections from:
  - `examples/typescript/README.md` (setup, structure, run, private key note)
  - `examples/python/legacy/README.md` (category breakdown)
- After merging, consider adding links to Java examples from relevant docs pages.

## Next Steps
1. Scaffold `examples/java/README.md` and the first two examples (`clients/httpclient`, `servers/jetty-servlet`).
2. Implement minimal `CryptoSigner` for EVM exact within the client example.
3. Verify local build flow: install SDK, run examples.
4. Open PR 1 for review; iterate.
