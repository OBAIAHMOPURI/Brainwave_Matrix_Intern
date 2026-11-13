# Banky Application — Architecture

[README](../README.md) · [Setup](SETUP.md) · [Network Ecosystem](../README.md#network-ecosystem)

## Scope and evidence

Based on static inspection of `main` on 13 November 2026, including the POM, all eleven Java handlers, deployment descriptor, and user-facing forms/menus. This is a source-level description, not a runtime or security certification. Proposed remediation is explicitly separated from current behavior.

## Current components and data flow

1. Static HTML forms under `Banky_Application/src/main/webapp/` collect request parameters.
2. `javax.servlet.http.HttpServlet` implementations map case-sensitive POST routes with `@WebServlet`.
3. Each handler directly opens a MySQL connection through `DriverManager`, prepares SQL, binds parameters, and reads or writes database rows.
4. Responses are static-page forwards/includes via `RequestDispatcher` or direct `PrintWriter` output. No templating/controller-service-DAO separation was found.

The main entry menu is `SelectOptions.html`; it links admin and account login pages. `OptionsForAdmin.html` links account creation, lookup/listing, and profile update. `OptionsForAccount.html` links deposit, balance check, and withdrawal. `TransferFunds.html` is present, but is not a functioning transfer flow. The root `index.jsp` is a Hello World stub.

## Route and parameter inventory

All routes below are relative to the WAR context and implement `doPost`, not a documented JSON REST API.

| POST route | Handler source | Request parameter names |
| --- | --- | --- |
| `/AddAccountInfo` | [AddAccountInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/AddAccountInfo.java) | `id`, `accountnumber`, `pincode`, `balance`, `name`, `mobilenumber` |
| `/AddAdmin` | [AddAdminInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/AddAdminInfo.java) | `id`, `email`, `password`, `name` |
| `/CheckBalance` | [CheckBalanceInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/CheckBalanceInfo.java) | `accountnumber`, `pincode` |
| `/DepositMoney` | [DepositMoneyInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/DepositMoneyInfo.java) | `accountnumber`, `balance` |
| `/LoginAccountDetails` | [LoginAccountDetails.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/LoginAccountDetails.java) | `accountnumber`, `pincode` |
| `/LoginAdminDetails` | [LoginAdminDetails.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/LoginAdminDetails.java) | `email`, `password` |
| `/TransferFunds` | [TransferFudsInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/TransferFudsInfo.java) | `accountnumber`, `balance` |
| `/UpdateProfile` | [UpdatePorfileInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/UpdatePorfileInfo.java) | `id`, `mobilenumber` |
| `/ViewAccountByAccountNumber` | [ViewAccountByAccountNumberInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/ViewAccountByAccountNumberInfo.java) | `accountnumber` |
| `/viewAllAccounts` | [ViewAllAccountsInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/ViewAllAccountsInfo.java) | None |
| `/WithDrawBalance` | [WithDrawBalanceInfo.java](../Banky_Application/src/main/java/com/jsp/BankyApplication/WithDrawBalanceInfo.java) | `accountnumber`, `balance` |

The misspelled class filenames `TransferFudsInfo.java` and `UpdatePorfileInfo.java` are the actual committed names. `/viewAllAccounts` has a lower-case initial `v`. Do not silently normalize these paths in clients or documentation.

## Database contract inferred from SQL

Current source targets `jdbc:mysql://localhost:3306/bank`.

| Table | Columns referenced | Important behavior |
| --- | --- | --- |
| `account` | `id`, `accountnumber`, `pincode`, `balance`, `name`, `mobilenumber` | Registration, lookup, login, balance updates, mobile-number update |
| `admin` | `id`, `name`, `email`, `password` | Registration and direct email/password comparison |

No original DDL, migrations, seed fixture, or constraints are committed. Several reads use `SELECT *` and numerical result-set indexes; `account` must return columns in the order `id`, `accountnumber`, `pincode`, `balance`, `name`, `mobilenumber` for that code to match its assumptions. Data types, lengths, keys, and relational constraints are not established by the repository. Any sample schema in [Setup](SETUP.md) is a provisional disposable-demo reconstruction, not the original schema.

## Verified implementation limitations

- **Transfers:** `TransferFudsInfo` reads repeated parameter names, queries accounts, and advances a result set; it performs no debit, credit, commit, or balance transfer.
- **Withdrawals:** the initial balance query is unfiltered and checks its first row; the later update targets the submitted account. This does not establish sufficient balance for that account and is not concurrency-safe.
- **Profile changes:** `UpdatePorfileInfo` updates only `mobilenumber` by `id`. Other form fields are ignored.
- **Authentication/authorization:** login checks forward to menus without a persisted/enforced session. No common authentication filter or role checks protect the other servlet operations in the audited tree.
- **Sensitive output:** account lookup/listing prints the stored PIN; admin passwords are directly stored/compared and PINs are not hashed.
- **Input and monetary correctness:** numerical parsing can fail without a useful client error; amount positivity, account ownership, and robust transactional guarantees are absent. Floating-point monetary handling is unsuitable for a real banking system.
- **Resources/HTTP:** connection/result handling and response content types are inconsistent; several HTML forms contain duplicate `action` attributes.

## Deployment/configuration boundaries

The servlet source embeds database connection credentials in `DriverManager.getConnection(...)`. Their values are intentionally not reproduced here. Do not reuse them; rotate any credential that was ever valid outside a disposable local environment. Before running, replace embedded credentials with a reviewed environment/JNDI/DataSource configuration and a new least-privilege local database account. **The current source does not read `DB_URL`, `DB_USER`, `DB_PASSWORD`, or a `.env` file: setting variables alone will not configure it.** Configuration refactoring and credential rotation are separate, unperformed changes; this documentation pass modifies no Java or deployment configuration.

The committed `WEB-INF/web.xml` declares Web Application 2.3 while servlet mappings are declared with `@WebServlet`. Verify annotation discovery in the chosen container. If form POSTs return 404, inspect the application startup log and mappings; modernizing the descriptor to Servlet 4.0 or explicitly registering mappings may be required. This guide does not claim that this configuration change has been made.

The documented dependency stack is Servlet 4.0.1 with `javax.servlet`, MySQL Connector/J 8.0.33, and a WAR artifact. No container, cloud resources, active gateway, workflow, telemetry, or notification integration is included.

## Proposed next architecture — not implemented

Introduce centralized configuration and a least-privilege pooled DataSource; isolate persistence behind a DAO and business rules behind a service layer; use secure password/PIN handling, session/role enforcement, and CSRF controls; return validated safe responses. Use exact decimal monetary values and atomic account-specific transactions with concurrency protection and audit trails. Add schema migrations and synthetic fixtures. Decide API contracts before connecting any sibling service.

Regression tests should cover missing/invalid inputs, unauthorized direct access, credential/PIN disclosure, no-row lookups, account-specific withdrawals, negative/zero amounts, concurrent updates, rollback, and incomplete transfers. Do not advertise a transfer feature until those semantics are implemented and verified.

## Verification status

No build, database creation, Tomcat start, browser smoke test, vulnerability scan, or end-to-end test was executed during this documentation pass. The tree contains generated `target/` classes, but they are not evidence of a reproducible clean build. A JUnit 3.8.1 test dependency is declared; no `src/test` tree or CI workflow was found. A successful Maven test phase without test sources does not demonstrate application correctness.
