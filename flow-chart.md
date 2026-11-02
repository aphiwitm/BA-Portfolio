# Workflow Diagrams

These Mermaid diagrams describe the proposed workflows for the pre-development requirements baseline. They support business review and development handoff.

## 1. Label generation and daily documents

```mermaid
flowchart TD
    A([Receive price or promotion change]) --> B[Identify branch, product, and effective data]
    B --> C{Duplicate event?}
    C -->|Yes| D[Keep existing records; do not duplicate]
    C -->|No| E[Evaluate product eligibility]
    E --> F{Eligible?}
    F -->|No| G[Do not generate label]
    F -->|Yes| H[Select ranked promotion winner]
    H --> I[Add permitted unranked promotions]
    I --> J[Handle superseded label versions]
    J --> K[Create latest unprinted label items]
    K --> L[Append to branch daily promotion document]
    L --> M[Update date, totals, and pending count]
    M --> N[Notify store of new pending labels]
```

Ranked selection uses approved rank, longer total duration, then lower price. A complete tie produces one ranked label. Unranked promotion labels may coexist. Promotion-end events can generate the effective normal-price label after selection and eligibility checks.

## 2. Printing and recovery

```mermaid
flowchart TD
    A([Store employee starts ordinary printing]) --> B[Select unprinted items from own branch]
    B --> C{Another operator actively printing or retrying?}
    C -->|Yes| D[Wait; do not submit duplicate work]
    C -->|No| E[Validate latest eligibility and versions]
    E --> F{Any excluded items?}
    F -->|Yes| G[Popup: excluded items and reasons]
    G --> H{Employee decision}
    H -->|Cancel all| X([No submission])
    H -->|Print eligible only| I{Any eligible items?}
    F -->|No| I
    I -->|No| X
    I -->|Yes| J[Freeze round: items, versions, and copy quantities]
    J --> K[Submit round and hold branch printing access]
    K --> L{Whole-round outcome}
    L -->|Success| M[Mark submitted items printed]
    M --> N[Preserve history and release access when idle]
    L -->|Unknown| O[Hold round; central operations investigates]
    O --> P[Allow unrelated branch work when otherwise idle]
    P --> Q[Return verified result for original round]
    Q --> L
    L -->|Confirmed failure| R[Document failed; retain earlier successful rounds]
    R --> S{Automatic retry already used?}
    S -->|No| T[Wait 30 seconds; revalidate failed set]
    T --> U[Exclude obsolete or no-longer-eligible labels]
    U --> V{Any items still eligible?}
    V -->|No| W([Retain obsolete or suppressed history])
    V -->|Yes| Y[Retry remaining failed set once]
    Y --> L
    S -->|Yes| Z[Employee opens external printer-fault ticket]
    Z --> AA[Autofill prefix; employee enters numeric suffix]
    AA --> AB{Ticket exists, printer fault, same branch?}
    AB -->|No or API unavailable| AC[Keep unresolved; correct or recheck]
    AC --> AA
    AB -->|Yes| AD[Close affected failed work as Repair Reported]
    AD --> AE([Keep history; no automatic requeue])
```

Unknown outcomes do not consume a new retry budget merely through investigation. A retry never includes later arrivals or successful earlier rounds. If a round contains obsolete items and exhausted failures, ticket closure applies to the remaining failures independently of obsolete history.

## 3. Reprinting

```mermaid
flowchart TD
    A([Select previously printed labels]) --> B[Optional Select All within current branch]
    B --> C[Click Reprint; no reason required]
    C --> D[Validate branch authorization and latest product rules]
    D --> E{Label type}
    E -->|Promotion| F[Check unexpired and not actually superseded]
    E -->|Normal price| G[Check current price and text]
    F --> H{Any excluded selections?}
    G --> H
    H -->|Yes| I[Show excluded labels and reasons]
    I --> J{Print eligible only or cancel?}
    J -->|Cancel| K([No submission])
    J -->|Eligible only| L{Any eligible labels?}
    H -->|No| L
    L -->|No| K
    L -->|Yes| M[Submit a new round using the same printing controls]
```

Reprinting never reopens ticket-closed failed work. A future-effective promotion does not replace today's valid label before its effective time. Permitted coexisting promotion labels do not supersede one another simply because one arrived later.

## 4. Version changes during printing

```mermaid
flowchart LR
    A[New effective version arrives] --> B{Old version already submitted?}
    B -->|No| C[Cancel old unprinted version and retain history]
    B -->|Yes| D[Retain actual old-round outcome; flag obsolete]
    C --> E[Create latest version as unprinted]
    D --> E
    D --> F[Do not retry obsolete version]
    D --> G[Old callback cannot mark new version printed]
```

## Text fallback

Receive event → evaluate eligibility and promotions → append latest items to the daily document → notify → select and validate → confirm exclusions → submit a fixed round → success, central reconciliation, or one retry → ticket closure if retry fails. Earlier successful rounds remain unchanged throughout.

Related specification: [Requirements](requirement.md). Detailed acceptance expectations: [Functional Requirements](functional-request.md).

