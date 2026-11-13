# Banky Application

A Java Servlet/JDBC banking **learning application** located in [`Banky_Application/`](Banky_Application/). HTML forms submit to servlets that execute SQL against a local MySQL `bank` schema. This is an educational prototype, not a production banking system.

> **Safety/status:** Source exists, but end-to-end behavior has not been verified. Database DDL and runtime configuration are not supplied, database credentials are embedded in source, and several operations are incomplete or unsafe. Use synthetic data in an isolated local environment only.

## Features present in source

- Account and admin registration handlers and corresponding HTML forms.
- Account-number/PIN and admin-email/password login checks; these forward to static menus but do **not** establish an enforced authenticated session.
- Account balance lookup, deposit, withdrawal, lookup by account number, and listing of accounts.
- Profile update currently changes the mobile number only, despite the form presenting more fields.
- A transfer form and handler exist, but the handler only reads account data; **fund transfer is not implemented**.

These are source-observed code paths, not claims of tested or secure functionality.

## Stack and repository layout

| Item | Verified value |
| --- | --- |
| Packaging | Maven WAR, version `0.0.1-SNAPSHOT` |
| Servlet API | `javax.servlet-api:4.0.1`, provided by the container |
| JDBC driver | `com.mysql:mysql-connector-j:8.0.33` |
| Test dependency | JUnit `3.8.1`; no test sources found |
| WAR name | `Banky_Application.war` |
| Database URL in current source | `jdbc:mysql://localhost:3306/bank` |

```text
Banky_Application/
  pom.xml
  src/main/java/com/jsp/BankyApplication/   # Servlet handlers and direct JDBC
  src/main/webapp/                        # Forms, menus, status pages, index.jsp
  src/main/webapp/WEB-INF/web.xml          # Legacy deployment descriptor
  target/                                 # Committed generated artifacts; rebuild locally
```

## Architecture notes

```text
Browser HTML form -> HttpServlet.doPost -> JDBC PreparedStatement -> MySQL bank
                         |                                      |
                         +---- static forward/include or text <-+
```

There is no separate service/DAO layer, session-enforcement filter, connection pool, migration system, or implemented sibling-service integration in the audited tree. See [Architecture](docs/ARCHITECTURE.md) for exact routes, database assumptions, trust boundaries, and known defects.

## Setup instructions

```sh
git clone https://github.com/OBAIAHMOPURI/Brainwave_Matrix_Intern.git
cd Brainwave_Matrix_Intern/Banky_Application
```

Before building/running, review [Setup](docs/SETUP.md): it explains the suggested JDK/Maven/Tomcat/MySQL baseline, missing original schema, credential/configuration remediation, annotation-discovery risk, an unverified explicit-plugin WAR build, and isolated smoke checks. There is no verified one-command startup.

After those prerequisites and fixes are completed in your local environment, the intended menu path is `/Banky_Application/SelectOptions.html`; the root `index.jsp` is only a Hello World page. Host and port depend on your container configuration.

## Important limitations and security work

- The transfer handler performs no debit/credit transaction.
- Withdrawal checks the first account row instead of querying the requested account before subtracting funds.
- Login forwards do not protect subsequent routes; no session-based access control was found.
- Account listing/lookup renders PIN values; credentials/PINs are not properly protected.
- Deposits/withdrawals lack sufficient validation and transactional safeguards; money is parsed as floating point in Java.
- JDBC resource/error handling is inconsistent. Some HTML forms repeat the `action` attribute.

Do not expose this application to untrusted networks or use real financial/customer data. Documentation does not repair these issues.

## Network Ecosystem

This is a navigation network across all eight repositories owned by this account, audited on 13 November 2026. Links indicate sibling projects, **not implemented integrations**. The Java apps are independent learning projects; the six service repositories have no application code.

| Sibling repository | Role and verified maturity |
| --- | --- |
| [alert-notification-dispatcher](https://github.com/OBAIAHMOPURI/alert-notification-dispatcher) | Proposed alert routing and delivery; no implementation |
| [BrainWave_Matrix_Intern1](https://github.com/OBAIAHMOPURI/BrainWave_Matrix_Intern1) | Java Servlet/JDBC hospital learning app; incomplete and unverified |
| [edge-gateway-service](https://github.com/OBAIAHMOPURI/edge-gateway-service) | Proposed ingress/routing service; previously empty |
| [edge-gateway-service1](https://github.com/OBAIAHMOPURI/edge-gateway-service1) | Separate gateway scaffold; relationship to other gateway undecided |
| [identity-auth-proxy](https://github.com/OBAIAHMOPURI/identity-auth-proxy) | Proposed identity/policy boundary; no implementation |
| [telemetry-stream-worker](https://github.com/OBAIAHMOPURI/telemetry-stream-worker) | Proposed telemetry ingestion worker; no implementation |
| [workflow-state-sync](https://github.com/OBAIAHMOPURI/workflow-state-sync) | Proposed state reconciliation service; no implementation |

Both gateway repositories remain distinct. No canonical gateway, shared protocol, deployment, or runtime connection has been established. Do not infer a production microservices system from these cross-links.

## Development and verification

No build, database creation, Tomcat start, browser smoke test, vulnerability scan, or end-to-end test was executed during this documentation pass. The tree contains generated `target/` classes, but they are not evidence of a reproducible clean build. A JUnit 3.8.1 test dependency is declared; no `src/test` tree or CI workflow was found. A successful Maven test phase without test sources does not demonstrate application correctness.

For a future code change, prioritize credential removal/rotation, authentication/authorization, secret/PIN protection, correct account-specific transactional operations, validation, resource cleanup, migration scripts, build pinning, and regression tests. No license file was found in the audited tree; no license is added by this documentation.
