# Authenticated Web and API Security with OWASP ZAP

**A sign-in page is not the application. Test the authorized and denied paths behind it.**

[Business problem](#the-business-problem) · [Solution](#the-solution) · [Strengths](#engineering-strengths) · [Proof](#proof-it-works) · [Commands](docs/COMMANDS.md)

![Local implementation reference](docs/images/01-overview.png)

## The Business Problem

A workforce platform used Microsoft sign-in and role-protected APIs. An anonymous web scan could inspect the public shell yet never reach the routes where staff permissions mattered. Test tokens and response bodies also needed protection from reports.

## The Solution

Use a pinned ZAP image and a fixed passive ten-GET matrix for anonymous, Admin and Caregiver-only identities. Verify exact identity/role before protected probes, refuse redirects/arbitrary routes and summarize only fixed alert metadata.

**Business outcome:** Recorded nonproduction acceptance exercised ten GETs across four handlers, with two informational cache observations and no Low/Medium/High alerts in that bounded sample. It was not whole-application clearance.

## Engineering Strengths

| Strength | Practical value |
| --- | --- |
| **HTTP behavior** | Inspect real requests/responses, including headers and behavior that package/IaC scanners cannot observe. |
| **Authenticated coverage** | Reach explicitly selected paths after sign-in, then compare authorized and lower-privilege behavior. |
| **Controlled passive testing** | Observe selected traffic without injecting active attack payloads; combine exact role assertions with safe alert summaries. |

## Architecture

```mermaid
flowchart LR
  Browser[Fresh authorized sign-in] --> Token[Private short-lived tokens]
  Token --> Matrix[Fixed ten-GET matrix]
  Matrix --> ZAP[Pinned passive ZAP proxy]
  ZAP --> API[Reviewed API handlers]
  API --> Claims[Identity / role / expected status]
  ZAP --> Summary[Alert IDs / risk / counts only]
```

This project is derived from actual workforce-platform implementation. Identifying labels/targets use examples. The local viewer and demonstration are new reference code; [source mapping](docs/SOURCE-MAP.md) identifies the copied/extracted files and limitations.

## What Was Built

| Component | Purpose | Implementation |
| --- | --- | --- |
| Actual authenticated adapter | Fixed target, private token handling and alert-summary filtering | [Source](scripts/zap_authenticated.py) |
| Actual role matrix | Four allowlisted handlers and expected actor/status observations | [Source](scripts/zap_role_matrix.py) |
| Pinned ZAP image | Immutable selected scanner reference | [Pin](security/zap-lock.json) |
| Actual tests | Reject wrong identity, malformed roles, arbitrary routes and token/report leakage | [Tests](scripts/) |
| Local demonstration | Exercise actual rule/evaluation code using fictional data | [Demo](scripts/demo.py) |
| Dated context | Keep recorded implementation separate from new local evidence | [Context](data/history.json) |

## Proof It Works

### 1. Run focused checks

```bash
python -B -m unittest discover -s scripts -p 'test_*.py' -v
python -B scripts/demo.py
```

**Observed:** Nine original Python contract tests passed. The fictional probe exercises the actual ten-request evaluator. [Validation](docs/VALIDATION.md) records the exact scope and distinguishes pure/mock checks from hosted integration.

### 2. Inspect the decision output

![Working reference example](docs/images/02-example.png)

The output is explicitly fictional and exercises the included logic. A successful rule/mock result is not a completed real-world operation. No new hosted scan, token collection or Microsoft sign-in was performed. The public target is deliberately api.workforce.invalid.

### 3. Open the reference viewer

```bash
python -B scripts/preview.py --port 8139
```

Open [the local reference](http://127.0.0.1:8139). Switch between Overview, Working example and Implementation context. Ctrl+C stops it. Screenshots are genuine local captures, not hosted application/Azure Portal screenshots.

## Passive, Active and Authenticated

Passive scanning inspects the traffic sent through ZAP. Active scanning generates attack requests and requires a separately authorized target/scope. Authentication changes which paths can be reached; it does not make every scan authenticated or give ZAP permission to test business writes.

The implemented adapter sends only a fixed set of GETs, with no spider, active scan or Microsoft-login replay. It verifies two distinct approved identities and a Caregiver-only role set before protected probes. A second Admin account cannot prove Caregiver denial. Request status assertions are code-defined checks; ZAP alert counts are separate passive observations.

The invalid query/person probes are deliberately expected to return specific Admin error statuses after authorization (400/503), while anonymous and Caregiver calls must be denied (401/403). Those Admin errors are not proof of a successful business operation. This sample does not retrieve real business records or validate every PDF/route.

[ZAP authentication documentation](https://www.zaproxy.org/docs/desktop/start/features/authentication/) describes broader authentication options. The selected platform mode uses privately supplied short-lived tokens rather than automating passwords/MFA.


## Operations, Limits and Growth

Keep the target, owning state/identity and evidence boundary explicit. Validate expected failures as well as success; do not let an incomplete result become a green indicator. Preserve private inputs and unrelated changes. Proposed scaling needs measured bottlenecks, cost review, downstream limits and repeated denied/recovery checks. [Operations and scaling](docs/OPERATIONS-AND-SCALING.md).

## Project Guides

| Guide | Purpose |
| --- | --- |
| [Implementation](docs/IMPLEMENTATION.md) | Rules, boundaries and actual source design |
| [Commands](docs/COMMANDS.md) | Local tests, expected output and inspection |
| [Operations and scaling](docs/OPERATIONS-AND-SCALING.md) | Strengths, weaknesses, costs and growth |
| [Source map](docs/SOURCE-MAP.md) | Actual files and public adaptations |
| [Validation](docs/VALIDATION.md) | Observed local evidence and remaining integration scope |

No Azure service, business record or live state was changed. The public validation workflow is manual-only and was not dispatched, conserving hosted Actions minutes.
