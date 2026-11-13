# Contributing to Banky Application

[Root README](README.md) · [Architecture](docs/ARCHITECTURE.md) · [Setup](docs/SETUP.md) · [API guide](docs/API.md) · [Network Ecosystem](README.md#network-ecosystem)

## Scope and safe development

Banky is a Java Servlet/JDBC learning application under `Banky_Application/`, not a production banking service. Use an isolated local container/database and synthetic accounts only. Source presence and committed `target/` classes do not prove a clean build, correct transfers, or secure authentication.

Read setup before installing tools or running the app. A candidate JDK/Maven/Tomcat baseline and disposable schema sketch are documented there but remain unverified. Database credentials are embedded in source; do not reproduce or reuse their values. Setting `DB_*` variables alone does not configure the unchanged application.

```sh
git clone https://github.com/OBAIAHMOPURI/Brainwave_Matrix_Intern.git
cd Brainwave_Matrix_Intern
git switch -c fix/describe-change
```

Use focused branches and pull requests for normal contributions. This documentation batch's explicit direct-to-main authorization is not a standing merge rule. Build commands must be run from `Banky_Application/`, not the repository root.

## Highest-priority improvements

1. Remove embedded credentials through reviewed configuration/DataSource changes and rotate any credential reused outside a disposable environment.
2. Add authenticated sessions, role/ownership checks, CSRF protections, safe PIN/password handling, and redacted output; current login forwards do not protect later routes.
3. Correct account-specific withdrawals and implement atomic debit/credit transfers with concurrency/rollback tests. The current transfer handler does not transfer money.
4. Validate amounts and inputs, use exact monetary semantics, handle missing records, and consistently close JDBC resources.
5. Add reviewed migrations/fixtures, pin compatible compiler/build plugins, verify annotation discovery with the legacy descriptor, and introduce meaningful tests.

Keep actual class/route spellings until changes are coordinated: `TransferFudsInfo.java`, `UpdatePorfileInfo.java`, and `/viewAllAccounts` are current names. Profile updates currently change only the mobile number. Do not describe unfinished features as working.

## Required evidence for a pull request

Explain the problem, change scope, compatibility impact, test data, commands actually run, results, remaining risks, and rollback/migration needs. Add regression tests for touched behavior: unauthorized direct access, PIN disclosure, invalid/negative amounts, no-row lookups, account-specific withdrawals, concurrent updates, transfer atomicity, and failure recovery.

JUnit 3.8.1 is declared, but no test-source tree or verified CI suite was found. A test phase with zero tests is not coverage. Mark unexecuted builds/tests honestly. Do not commit new generated `target/` output; existing generated files are not authorization for unrelated cleanup.

For documentation, run `git diff --check`, inspect rendering and exact parameter casing, and check root README, architecture/setup/API, and all seven sibling links. Keep proposed remediation distinct from current behavior.

## Security and ownership

Never put real account data, PINs, passwords, credentials, private notes, or raw sensitive logs in issues, commits, or screenshots. Use an owner-approved private channel for sensitive reports. Production changes, deployment, account changes, and external integrations require separate approval. No license is selected by this guide.

## Sibling repository network

Navigation only; these are not working service integrations.

- [alert-notification-dispatcher](https://github.com/OBAIAHMOPURI/alert-notification-dispatcher/blob/main/README.md)
- [BrainWave_Matrix_Intern1 — Hospital Management System](https://github.com/OBAIAHMOPURI/BrainWave_Matrix_Intern1/blob/main/README.md)
- [edge-gateway-service](https://github.com/OBAIAHMOPURI/edge-gateway-service/blob/main/README.md)
- [edge-gateway-service1](https://github.com/OBAIAHMOPURI/edge-gateway-service1/blob/main/README.md)
- [identity-auth-proxy](https://github.com/OBAIAHMOPURI/identity-auth-proxy/blob/main/README.md)
- [telemetry-stream-worker](https://github.com/OBAIAHMOPURI/telemetry-stream-worker/blob/main/README.md)
- [workflow-state-sync](https://github.com/OBAIAHMOPURI/workflow-state-sync/blob/main/README.md)
