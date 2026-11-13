# Banky Application — API and Form Route Guide

[Root README](../README.md) · [Architecture](ARCHITECTURE.md) · [Setup](SETUP.md) · [Contributing](../CONTRIBUTING.md) · [Network Ecosystem](../README.md#network-ecosystem)

## Contract status and transport

This is a source-derived reference for the existing Java learning application's HTML-form handlers, audited on 13 November 2026. It is not a versioned JSON REST API or a claim of tested banking functionality. No deployed base URL, OpenAPI specification, client SDK, or health endpoint is provided.

Routes below are relative to the deployed WAR context, conventionally `/Banky_Application` when the WAR retains its configured name. Actual host, port, context, and annotation discovery depend on local container setup. The entry menu is `SelectOptions.html`; `index.jsp` is only Hello World. Follow setup before attempting requests.

All listed handlers implement `doPost`. Existing forms submit URL-encoded parameters (`application/x-www-form-urlencoded`); these are not JSON bodies. Route and field casing matter. Responses are static HTML forwards/includes or direct text/HTML output, not a consistent JSON/error envelope. No stable HTTP error-status, pagination, idempotency, or transaction guarantee is established by the current code.

## Exact POST route inventory

| Route | Parameters consumed by handler | Current source behavior and limits |
| --- | --- | --- |
| `/AddAccountInfo` | `id`, `accountnumber`, `pincode`, `balance`, `name`, `mobilenumber` | Inserts into `account`, then forwards to `File1.html` |
| `/AddAdmin` | `id`, `email`, `password`, `name` | Inserts into `admin`, then includes `File3.html`; password handling is unsafe |
| `/CheckBalance` | `accountnumber`, `pincode` | Queries matching account and prints account number/balance; missing-row handling is unsafe |
| `/DepositMoney` | `accountnumber`, `balance` | Adds the submitted amount to balance and forwards to `File4.html`; validation/authorization are insufficient |
| `/LoginAccountDetails` | `accountnumber`, `pincode` | Checks a matching row and forwards to `OptionsForAccount.html`; no enforced authenticated session is created |
| `/LoginAdminDetails` | `email`, `password` | Directly compares credentials and forwards to `OptionsForAdmin.html`; no role/session enforcement follows |
| `/TransferFunds` | `accountnumber`, `balance` | Reads the same parameter names twice and queries accounts; no debit or credit is executed |
| `/UpdateProfile` | `id`, `mobilenumber` | Updates mobile number only; other displayed form fields are ignored |
| `/ViewAccountByAccountNumber` | `accountnumber` | Prints matching account fields, including the stored PIN; unsafe disclosure |
| `/viewAllAccounts` | None | Prints all account rows, including PINs; no pagination or enforced access control |
| `/WithDrawBalance` | `accountnumber`, `balance` | Checks the first row from an unfiltered account query, then subtracts from the requested account; not financially correct or concurrency-safe |

The actual source filenames and links are indexed in [Architecture](ARCHITECTURE.md). Preserve the lower-case initial `v` in `/viewAllAccounts`; the actual classes include the spellings `TransferFudsInfo` and `UpdatePorfileInfo`.

## Input interpretation

Handlers parse `id` and `pincode` as Java integers; account/phone numbers as longs; and monetary `balance` values as doubles. These describe current parsing, not a recommended data model or validated constraints. Leading-zero semantics, positive amounts, account ownership, precision, and concurrency require remediation. Strings such as name/email/password have no comprehensive validation contract. The transfer form repeats account/amount field names and does not establish distinct sender/recipient semantics.

A schematic read-only balance request after safe local setup is:

```http
POST /Banky_Application/CheckBalance
Content-Type: application/x-www-form-urlencoded

accountnumber=<synthetic_account_number>&pincode=<disposable_pin>
```

Angle-bracket values are explanatory placeholders, not literal inputs or real credentials. Use only synthetic values and normal URL encoding. This documentation did not execute the request. Do not call write routes against real accounts; the implementation is unsafe even when a success page appears.

## Authentication, errors, and persistence caveats

Login forwarding is not an authorization boundary. No common session/role/ownership filter was found. PINs/passwords are not appropriately protected, and account views disclose PINs. Do not expose the app publicly. SQL/driver errors are generally logged, numerical parsing may fail unchecked, and an HTML response is not proof of correct database effects. Direct browser navigation uses GET and may produce 405; missing mappings may produce 404, particularly with the legacy descriptor/annotation-discovery mismatch.

The database target in source is local schema `bank`. Original DDL, migrations, and seed data are absent; the setup sketch is provisional. Embedded credential values are deliberately omitted here. No automatic database setup, build, runtime test, transaction repair, or sibling integration was performed by this documentation change.

## Future API work — not implemented

Add secure authentication/authorization, protected secrets, validated typed input, exact decimal monetary handling, atomic account-specific transactions, consistent status/error schemas, migration-backed fixtures, and meaningful regression tests before offering a programmatic API. Version and coordinate any renamed route/field with its HTML form and clients. Never advertise `/TransferFunds` as working until implemented and tested.

## Sibling repository network

Navigation only; no runtime connections are implied.

- [alert-notification-dispatcher](https://github.com/OBAIAHMOPURI/alert-notification-dispatcher/blob/main/README.md)
- [BrainWave_Matrix_Intern1 — Hospital Management System](https://github.com/OBAIAHMOPURI/BrainWave_Matrix_Intern1/blob/main/README.md)
- [edge-gateway-service](https://github.com/OBAIAHMOPURI/edge-gateway-service/blob/main/README.md)
- [edge-gateway-service1](https://github.com/OBAIAHMOPURI/edge-gateway-service1/blob/main/README.md)
- [identity-auth-proxy](https://github.com/OBAIAHMOPURI/identity-auth-proxy/blob/main/README.md)
- [telemetry-stream-worker](https://github.com/OBAIAHMOPURI/telemetry-stream-worker/blob/main/README.md)
- [workflow-state-sync](https://github.com/OBAIAHMOPURI/workflow-state-sync/blob/main/README.md)
