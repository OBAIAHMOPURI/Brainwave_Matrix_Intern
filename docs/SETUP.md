# Banky Application — Setup

[README](../README.md) · [Architecture](ARCHITECTURE.md)

## 1. Scope and prerequisites

This guide is for a **disposable, isolated learning environment with synthetic data only**. It is not a production deployment recipe. Known source defects and security gaps are documented in [Architecture](ARCHITECTURE.md).

A suggested local baseline is a JDK 11 installation, Maven 3.9.x, a patched Tomcat 9.x container compatible with `javax.servlet`/Servlet 4.0, and a disposable local MySQL 8.x database. This combination is a **proposal, not a tested compatibility matrix**. Select supported patched versions for your environment. The POM does not pin Java compiler settings or build-plugin versions. Tomcat 10+ uses `jakarta.servlet`; it is not a drop-in runtime for these unchanged `javax.servlet` imports.

Install Git and your selected tools, then inspect versions:

```sh
git --version
java -version
mvn -version
```

These checks and all commands below are instructions for future local execution; they were not run in this documentation pass.

## 2. Obtain the source

```sh
git clone https://github.com/OBAIAHMOPURI/Brainwave_Matrix_Intern.git
cd Brainwave_Matrix_Intern/Banky_Application
```

The Maven project is inside `Banky_Application/`, not at the repository root. Rebuild from source rather than relying on committed `target/` output.

## 3. Resolve configuration and schema blockers first

The servlet source embeds database connection credentials in `DriverManager.getConnection(...)`. Their values are intentionally not reproduced here. Do not reuse them; rotate any credential that was ever valid outside a disposable local environment. Before running, replace embedded credentials with a reviewed environment/JNDI/DataSource configuration and a new least-privilege local database account. **The current source does not read `DB_URL`, `DB_USER`, `DB_PASSWORD`, or a `.env` file: setting variables alone will not configure it.** Configuration refactoring and credential rotation are separate, unperformed changes; this documentation pass modifies no Java or deployment configuration.

The current JDBC URL is `jdbc:mysql://localhost:3306/bank`. Confirm host, port, schema, driver visibility, and connection privileges in your isolated environment. Do not grant administrative database privileges to the application.

No original database schema is provided. Obtain a reviewed schema if available. The following is an **illustrative, untested, disposable-only compatibility sketch**, derived from SQL column names and Java bindings. Keys, uniqueness, string lengths, and precise SQL types are proposed, not recovered facts. It is not a financial data model and lacks a ledger, transfer transaction design, and complete integrity checks. It must not be applied to an existing or production database.

```sql
-- ILLUSTRATIVE DISPOSABLE-DEMO SCHEMA, NOT THE ORIGINAL DATABASE DDL.
-- Review keys, lengths, types and constraints before using even in a demo.
CREATE DATABASE IF NOT EXISTS bank;
USE bank;
CREATE TABLE account (
  id INT PRIMARY KEY,
  accountnumber BIGINT NOT NULL UNIQUE,
  pincode INT NOT NULL,
  balance DECIMAL(19,2) NOT NULL,
  name VARCHAR(255) NOT NULL,
  mobilenumber BIGINT NOT NULL
);
CREATE TABLE admin (
  id INT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL
);
```

Account-column order is significant because current source uses numerical indexes after `SELECT *`. Current Java represents account/phone numbers as numeric values and PINs as integers, which loses leading-zero semantics. Although the sketch uses SQL `DECIMAL`, Java still uses `double`; the sketch does not fix monetary correctness. The `admin.password` column reflects the insecure current plaintext comparison and must only receive disposable fake credentials; redesign and hash secrets before any real use. No seed passwords are supplied here.

## 4. Review servlet discovery

The committed `WEB-INF/web.xml` declares Web Application 2.3 while servlet mappings are declared with `@WebServlet`. Verify annotation discovery in the chosen container. If form POSTs return 404, inspect the application startup log and mappings; modernizing the descriptor to Servlet 4.0 or explicitly registering mappings may be required. This guide does not claim that this configuration change has been made.

Inspect exact annotation routes in [Architecture](ARCHITECTURE.md). Browser navigation to a POST-only servlet may produce HTTP 405 even when mapping exists; load the HTML page and submit its form instead.

## 5. Suggested WAR build — not verified

The POM does not pin compiler or WAR-plugin versions. A candidate local build on the suggested baseline explicitly chooses Java 8 source/target compatibility and a modern WAR plugin without editing the POM:

```sh
mvn -Dmaven.compiler.source=8 -Dmaven.compiler.target=8 clean test org.apache.maven.plugins:maven-war-plugin:3.4.0:war
```

Expected artifact if the build succeeds: `target/Banky_Application.war`. This command does not fix Java/database/security defects and is **not a claim of a successful build**. The `test` phase currently has no repository test sources. If compiler/plugin compatibility fails, inspect the full error and pin supported build configuration in a separate reviewed code/configuration change; do not use precompiled files as proof of correctness.

## 6. Isolated container deployment — after local remediation

Configure an isolated Tomcat 9 instance and its `CATALINA_HOME`/`CATALINA_BASE` for your operating system. After a successful local build, deploy the WAR through your approved local container mechanism. A Unix-like example is:

```sh
cp target/Banky_Application.war "$CATALINA_BASE/webapps/"
"$CATALINA_HOME/bin/catalina.sh" run
```

For a container configured on local port 8080, the illustrative menu URL is:

```text
http://localhost:8080/Banky_Application/SelectOptions.html
```

Adjust host, port, and context to your actual container. The root JSP is Hello World, not the app menu. Do not expose the container publicly.

## 7. Synthetic smoke checks and known expected failures

- Confirm the menu and form pages render and that servlet annotations are discovered.
- With fake identities only, test registration and login. A successful login response does not prove later routes are protected.
- Inspect database rows after account creation, deposit, and mobile-number update; record actual behavior rather than trusting a success page.
- Balance lookup on an unknown account may fail because it advances a result set without checking whether a row exists.
- Do **not** treat withdrawal as financially correct: it checks the wrong source row and lacks safe concurrency behavior.
- Do **not** expect a transfer to change balances: transfer logic is unfinished.
- Treat visible PINs and directly accessible privileged routes as security defects, not valid production behavior.

## 8. Troubleshooting

| Symptom | Evidence to check |
| --- | --- |
| Root page says Hello World | Open `SelectOptions.html` within the WAR context |
| Form POST returns 404 | WAR context, exact route case, deployment logs, legacy descriptor/annotation discovery |
| Direct route navigation returns 405 | Handler implements POST; submit its form |
| Unknown database/table or column errors | Missing original schema, configured database, exact `account` column contract/order |
| Database access denied | New local account permissions and reviewed configuration; do not reuse embedded secrets |
| Driver class not found | Built WAR dependencies and Connector/J packaging |
| No transfer effect | Known incomplete implementation, not a documentation/install issue |
| Profile changes other than mobile are ignored | Current SQL updates `mobilenumber` only |

## 9. Verification record

No build, database creation, Tomcat start, browser smoke test, vulnerability scan, or end-to-end test was executed during this documentation pass. The tree contains generated `target/` classes, but they are not evidence of a reproducible clean build. A JUnit 3.8.1 test dependency is declared; no `src/test` tree or CI workflow was found. A successful Maven test phase without test sources does not demonstrate application correctness.

Record actual tool versions, successful build command, schema revision, configured servlet mappings, and observed smoke-test results before calling this setup verified. Keep secrets out of logs, screenshots, GitHub, and memory checkpoints.
