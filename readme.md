# Retail Price and Promotion Label Management

A business analysis portfolio case study on reducing unnecessary retail label printing through eligibility rules, daily promotion documents, and controlled print recovery.

## Business problem

The existing printing process produces more labels than stores need. Store employees manually sort and discard unnecessary labels, consuming time and paper. The proposed workflow filters labels before printing and tracks which items have already been printed.

## Proposed solution

- Apply product eligibility rules before generating and printing labels.
- Group promotion labels into one document per branch per day.
- Append intraday additions to the same document and print only unprinted items.
- Treat each submitted print round as a single success, failure, or unknown outcome.
- Preserve successful earlier rounds when a later round fails.
- Support validated reprints, one automatic retry, central investigation, and ticket-based closure.

## Portfolio artifacts

| Artifact | Purpose |
|---|---|
| [Requirements](requirement.md) | Business objective, scope, actors, rules, data, assumptions, and acceptance scenarios |
| [Flowcharts](flow-chart.md) | Mermaid diagrams for generation, printing, recovery, and reprinting |
| [Functional Requirements](functional-request.md) | 24 linked user stories, functional requirements, and expected results |

GitHub renders the Mermaid diagrams directly in Markdown. Open the files in the order above to follow the analysis from business need to expected behavior.

## Business analysis work demonstrated

Requirements elicitation and clarification; business-rule analysis; workflow modeling; exception handling; acceptance criteria; and traceability through stable user story and requirement IDs.

The case study distinguishes a **daily document**, an **item**, a **print round**, and a **copy**. This distinction prevents a failed later submission from incorrectly resetting earlier successful items.

## Example

A daily promotion document contains 10 items. Six were printed successfully earlier. Four new items are submitted in a later round, which fails. The document shows a failure, but the original six remain successful. Only the four items in the failed round are eligible for its automatic retry, subject to current validity checks.

## Scope and validation status

This repository contains English portfolio documentation. The associated project has an offline HTML prototype with synthetic data and simulated upstream, printing, and ticket-validation integrations; that prototype is not included in this documentation package.

Developer logic checks were performed and defects were corrected. The 53 browser-interaction QA cases remain blocked and testing is paused. End-to-end browser QA and UAT are not complete. No production deployment or measured business savings are claimed.

## Intended benefits

Less unnecessary printing, less manual sorting, clearer pending-work visibility, and traceable recovery from printing failures. These are expected benefits, not measured outcomes.

## Publication notes

Branch IDs, product codes, prices, ticket numbers, and dates used in examples are illustrative. The documentation contains no credentials, private API endpoints, or local machine paths. Real integration contracts and production access controls must be defined separately.

Documentation baseline: consolidated Requirement v2.0, including daily document processing, reprinting without a reason, Select All, and automatic ticket prefixes.
