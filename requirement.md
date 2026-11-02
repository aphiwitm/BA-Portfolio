# Requirements Specification

## 1. Objective

Reduce unnecessary price and promotion label printing and store employees' manual sorting effort. Generate eligible labels, display pending work, and maintain accurate printing history without silently reprinting successful earlier work.

## 2. Scope

**In scope:** product eligibility, promotion prioritization, event-based generation, daily promotion documents, notifications, item selection, ordinary printing, reprinting, round-level results, retry, central reconciliation, ticket validation, and history.

**Outside the current prototype scope:** live integrations, physical printer repair, upstream product-data maintenance, production authentication, and creating replacement jobs after repair for ticket-closed work.

The offline prototype simulates integrations. Production functional requirements describe the required behavior, not evidence that those integrations already exist or have been tested.

## 3. Actors

| Actor | Responsibility |
|---|---|
| Store employee | Print and reprint own-branch labels; raise a printer-fault ticket externally and enter its numeric suffix |
| Central operations | Investigate unknown print outcomes and return verified results |
| Business team | Approve promotion ranking data |
| Upstream systems | Supply products, prices, promotions, change events, and placement-based copy quantities |
| Printing system | Return an outcome correlated to the submitted print round |
| Ticketing system | Verify ticket existence, printer-fault category, and branch |

## 4. Domain definitions

| Term | Definition |
|---|---|
| Daily promotion document | One branch's promotion-label collection for a document date |
| Item | An individual label version in a document |
| Print round | A fixed set of items and copy quantities submitted together |
| Copy | A physical label quantity supplied by upstream placement data; not the document item count |
| Superseded version | A label version replaced by newer effective information |

New arrivals can extend a document, but cannot silently extend a round already submitted.

## 5. Business rules

### 5.1 Product eligibility

Apply all applicable conditions together, with unconditional exclusions taking precedence.

| Rule ID | Condition | Required behavior |
|---|---|---|
| BR-01 | Sale is not allowed | Exclude in every case |
| BR-02 | Location is `99` | Exclude in every case |
| BR-03 | Location is `00` | Require `stock_qty > 0` and all other rules |
| BR-04 | Product is inactive | Require sale permission, `stock_qty > 0`, and all other rules |
| BR-05 | Assortment is false | Require `stock_qty > 0` and all other rules |
| BR-06 | Active, assortment true, valid location `01–10` | Zero stock alone does not exclude; sale permission and other applicable rules still apply |

Upstream systems own the correctness of location and stock data. Eligibility must be checked again immediately before printing and reprinting.

### 5.2 Generation and promotion selection

| Rule ID | Requirement |
|---|---|
| BR-07 | Generate on promotion start/end, price changes, or label-text changes, including intraday updates. A new calendar day alone does not regenerate identical labels. |
| BR-08 | For overlapping ranked promotions, use the business-approved ranking table; for equal rank, prefer longer total start-to-end duration, then lower promotional price. If rank, duration, and price tie, produce one label. |
| BR-09 | Generate unranked promotion labels additionally, even if a ranked promotion label already exists. All product eligibility rules still apply. |
| BR-10 | Cancel a superseded unsubmitted label version, keep its history, and create the latest version as unprinted. Duplicate delivery of the same event must not create duplicate work. |

Production ranking values and order are supplied through the approved table. Demonstration values are not business policy.

### 5.3 Documents and printing

| Rule ID | Requirement |
|---|---|
| BR-11 | Maintain one promotion document per branch per day; show date, total items, successful items, pending items, and document status. |
| BR-12 | Append intraday arrivals to the same daily document. Ordinary printing submits only eligible unprinted items. |
| BR-13 | Select all active-branch rows by default; allow deselection and an explicit Select All action, including for reprint preparation. Selection never overrides eligibility. |
| BR-14 | Show excluded items and reasons in a popup. Require confirmation to print eligible items only, or cancel everything. Submit nothing if none qualify. |
| BR-15 | Freeze items, versions, and upstream copy quantities when a round is submitted. Receive one outcome for the entire submitted round. Submission alone does not mean success. |
| BR-16 | A failed round marks its submitted items failed and the document as failed. Earlier successful rounds remain successful. |
| BR-17 | Allow one active printing operator per branch, including active retries. Held unknown outcomes do not block unrelated work once no active printing/retry remains. |

### 5.4 Recovery and reprinting

| Rule ID | Requirement |
|---|---|
| BR-18 | Retry a confirmed failed round once after 30 seconds: at most two submissions including the initial attempt. Do not include successful earlier rounds or later arrivals. Recheck current validity and suppress obsolete versions. |
| BR-19 | Hold unknown outcomes for central investigation; do not automatically resubmit. Reconciliation does not reset the retry budget. |
| BR-20 | After retry exhaustion, the employee raises an external printer-fault ticket. Autofill the ticket prefix; accept only the numeric suffix; compose the full identifier for validation. |
| BR-21 | Close failed work only after the ticket exists, has the correct fault category, and matches the branch. API unavailability or failed validation prevents closure. |
| BR-22 | One validated ticket may close multiple failed items from the same fault incident and branch. Preserve history; close as Print Failed—Repair Reported, never Printed. Do not automatically requeue or create post-repair replacements. |
| BR-23 | Reprints require own-branch authorization and current eligibility, but no reason entry. Promotion labels must be unexpired and not actually superseded; future-effective updates and permitted coexisting promotions do not automatically block them. Normal-price labels must retain the current price/text. |
| BR-24 | Keep actual outcomes of superseded in-flight versions separate from new versions. An old result cannot mark a new label printed. Never retry an obsolete version. |
| BR-25 | Notify immediately when new promotion work is received while open; keep the message until printing is initiated. On reopening, display pending current work again. |

## 6. Data and integration dependencies

Required product fields: `branch`, `barcode`, `is_allow_sale`, `is_active`, `assortment`, `location`, and `stock_qty`.

Additional information includes promotion identity, effective start/end timestamps, price/text version, approved ranking table version, placement-based copy count, document identity/date, round identity, operator, outcome, and validated ticket reference.

Before production implementation, define source schemas, event identity and delivery contracts, document-date/time-zone rules, round-result correlation, actual ticket-prefix format, and how the same printer incident is established. The prototype uses illustrative values; no private integration details are included here.

## 7. Key acceptance scenarios

| Scenario | Expected result |
|---|---|
| Sale prohibited, location `00`, positive stock | No label: sale prohibition overrides stock |
| Active, assortment true, location `01`, zero stock | Eligible if sale is allowed and other requirements hold |
| Six items successful; four new items fail | Document failed; six retained as successful; only the failed four belong to the retry |
| New items arrive while a round is active | Add to the daily document, leave pending for a subsequent round |
| A future promotion starts tomorrow | Today's valid unexpired label remains eligible for reprint |
| Retry fails, ticket validation succeeds | Close affected failed work with its ticket; preserve earlier successes; no automatic requeue |
| Earlier successful job updates after ticket closure of a later job | Closed status remains closed; historical results cannot overwrite the latest job state |

## 8. Validation status

53 browser QA cases were prepared but remain blocked and testing is paused. Developer logic checks are separate evidence, not browser QA or UAT sign-off. No production acceptance or measured benefits are asserted.
