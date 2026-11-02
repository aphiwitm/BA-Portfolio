# Functional Requirements and User Stories

Each entry links a user story (`US`), functional requirement (`FR`), and expected result (`ER`) by the same number. Expected results define the required behavior of the proposed system and guide implementation.

## Generation and preparation

### FR-01 — Event-based generation

**US-01:** As a store employee, I want labels prepared when relevant information changes so that I do not create print work manually.

**Functional request:** Process promotion start/end, price/text changes, and intraday additions. Do not regenerate identical labels solely because the date changes.

**ER-01:** A qualifying change creates the current eligible version as unprinted; an unchanged day creates no duplicate labels.

### FR-02 — Product eligibility

**US-02:** As a store employee, I want ineligible products excluded so that fewer unnecessary labels reach the printer.

**Functional request:** Apply BR-01–06 in [Requirements](requirement.md), both at generation and immediately before submission.

**ER-02:** Sale prohibition and location `99` always exclude. Valid active assortment products may qualify at zero stock; the conditional positive-stock rules remain enforced.

### FR-03 — Promotion prioritization

**US-03:** As a business stakeholder, I want overlapping promotions selected using approved rules so that label selection is consistent.

**Functional request:** Use approved rank, longer total duration, and then lower price; emit one ranked label for a complete tie. Include eligible unranked promotions additionally.

**ER-03:** The ranked winner follows that order; permitted unranked labels are not discarded merely because a ranked label exists.

### FR-04 — Daily promotion documents

**US-04:** As a store employee, I want one document per branch and day so that I can understand that day's promotion workload.

**Functional request:** Display document date, total item count, successful count, pending count, and status. Append same-day additions to that document.

**ER-04:** A document such as “11 Nov 2026 — 10 items” contains the relevant branch's items; copy quantity is not mistaken for item count.

### FR-05 — Persistent notifications

**US-05:** As a store employee, I want new pending promotion work announced so that I know when to print.

**Functional request:** Notify immediately on receipt while open, retain the message until printing is initiated, and restore pending-work visibility on reopening.

**ER-05:** An update received at 10:00 triggers a visible message. Reopening while current items remain pending displays the reminder again.

### FR-06 — Selection defaults and Select All

**US-06:** As a store employee, I want default selection and an explicit Select All option so that bulk printing and reprinting are convenient.

**Functional request:** Default-select the active branch's rows, allow deselection, and provide Select All. Keep selection consistent wherever the same item is displayed.

**ER-06:** Select All covers the current branch only; selection does not bypass print/reprint eligibility.

## Printing and results

### FR-07 — Print unprinted work only

**US-07:** As a store employee, I want ordinary printing to exclude completed items so that successful work is not printed again.

**Functional request:** Submit eligible unprinted items only. Distinguish “Print selected items” from the document-wide “Print all unprinted items” action.

**ER-07:** With six successful and four new items, ordinary printing submits only the four new items.

### FR-08 — Exclusion popup

**US-08:** As a store employee, I want to see exclusions and reasons before proceeding so that I understand which labels will not print.

**Functional request:** Show excluded labels and rule failures for ordinary printing and reprinting; offer Print eligible only or Cancel all.

**ER-08:** Confirmation submits only eligible selections; cancellation or zero eligible items produces no print submission.

### FR-09 — Fixed submission rounds

**US-09:** As a store employee, I want each round to have a fixed scope so that results and retries apply to the correct work.

**Functional request:** Capture round identity, label versions, selected items, and upstream copy quantities at submission.

**ER-09:** Later arrivals join the document but remain unprinted for a subsequent round; they are not added to the active submission.

### FR-10 — Whole-round outcomes

**US-10:** As a store employee, I want one result for the submitted set so that the round has an unambiguous outcome.

**Functional request:** Accept success, failure, or unknown for the complete round. Do not present independent per-item/per-copy outcome controls within that round.

**ER-10:** Success marks its items printed; failure applies to its full submitted set. Sending a command alone never marks success.

### FR-11 — Preserve earlier successes

**US-11:** As a store employee, I want previous successful rounds retained when a later round fails so that I do not waste labels.

**Functional request:** Separate document failure visibility from historical successful item outcomes.

**ER-11:** Six successful plus four later failed items yield a failed document with six still successful; only the failed four belong to the retry.

### FR-12 — One automatic retry

**US-12:** As a store employee, I want a controlled retry for a confirmed failure so that transient problems can recover without repeated manual clicks.

**Functional request:** Retry the failed set once after 30 seconds. Revalidate current eligibility/versions; omit obsolete work and never add later arrivals.

**ER-12:** At most two submissions occur, including the initial one. Successful earlier work is not resubmitted.

### FR-13 — Unknown-outcome reconciliation

**US-13:** As a central operator, I want to verify unknown results so that uncertain work is not printed twice.

**Functional request:** Hold unknown rounds; obtain verified printing-system results through central operations; retain the original retry budget.

**ER-13:** No automatic retry occurs while the outcome is unknown; the verified result updates the original round.

### FR-14 — Branch printing exclusivity

**US-14:** As a store employee, I want simultaneous submissions controlled so that employees cannot create duplicate active work.

**Functional request:** Allow one active printing operator per branch, including active retries. Release access when no active print/retry remains.

**ER-14:** Another operator cannot submit during active work. A held unknown round alone does not prevent unrelated branch work.

## Reprints and versions

### FR-15 — Reprint without a reason

**US-15:** As a store employee, I want to replace previously printed labels without filling in a reason so that reprinting is straightforward.

**Functional request:** Do not require a reason field or reason entry for reprinting. Apply branch, status, eligibility, and effective-price/promotion checks.

**ER-15:** Eligible reprints start without reason entry; excluded selections still appear in the validation popup.

### FR-16 — Current reprint content

**US-16:** As a store employee, I want only valid current labels available for reprinting so that old prices are not reused.

**Functional request:** Require unexpired, unsuperseded promotion labels and current normal price/text. Distinguish future updates and permitted coexistence from actual replacement.

**ER-16:** Tomorrow's promotion alone does not block today's valid label; an effective replacement does block the superseded version.

### FR-17 — Changes before submission

**US-17:** As a store employee, I want the latest unprinted version available so that outdated pending labels are not sent.

**Functional request:** Cancel superseded unsubmitted versions, preserve history, create the latest version as pending, and deduplicate repeated event delivery.

**ER-17:** An unprinted price revision leaves the latest version pending and the old one in history; replaying the same event does not duplicate it.

### FR-18 — Changes during printing

**US-18:** As a store employee, I want old results separated from new label versions so that status remains accurate.

**Functional request:** Preserve actual in-flight old-version outcomes, flag obsolete content, create the new pending version, and suppress obsolete retries.

**ER-18:** An old successful callback cannot mark the new version printed. Obsolete history does not prevent recovery of remaining current failed work.

## Ticket closure and transparency

### FR-19 — Fault escalation

**US-19:** As a store employee, I want a clear escalation path after retry exhaustion so that printing faults have an accountable resolution.

**Functional request:** Stop automatic retries and direct the employee to raise a printer-fault ticket in the external ticketing system.

**ER-19:** Work remains failed until valid closure; opening the ticket does not falsely mark it printed.

### FR-20 — Automatic ticket prefix

**US-20:** As a store employee, I want the ticket prefix filled automatically so that I only enter the numeric suffix.

**Functional request:** Display the configured branch-relevant prefix, accept digits only, and compose the full identifier before validation.

**ER-20:** Illustratively, suffix `01` plus `T-001-PRN-` becomes `T-001-PRN-01`. Changing branch updates the prefix and clears the old suffix.

### FR-21 — Ticket validation

**US-21:** As a store employee, I want the ticket verified so that failed work references the correct repair request.

**Functional request:** Validate ticket existence, printer-fault category, and matching branch through the ticket API.

**ER-21:** Invalid tickets or API unavailability prevent closure and permit correction/rechecking; only a positive result allows closure.

### FR-22 — Shared incident closure

**US-22:** As a store employee, I want one ticket to cover related failures so that I do not duplicate incident reporting.

**Functional request:** Allow one validated ticket for selected retry-exhausted failures from the same incident and branch; exclude unknown outcomes and obsolete-only history.

**ER-22:** Related failures close with the same reference while prior successes remain unchanged; unrelated branches/incidents are not closed together.

### FR-23 — Terminal closure and history

**US-23:** As an auditor, I want reliable history and terminal closure so that previous work cannot silently reactivate a failed job.

**Functional request:** Keep operator, items, versions, round outcomes, and ticket references. Ticket-closed work cannot auto-requeue or use successful-item reprint to reopen; post-repair replacement creation is outside scope.

**ER-23:** Closure remains Print Failed—Repair Reported even after historical callbacks or periodic refresh. New eligible work may still arise from normal price/promotion events.

### FR-24 — Integration contracts for development handoff

**US-24:** As a development team member, I want the required integration data and responsibilities defined so that interfaces can be designed consistently with the business workflow.

**Functional request:** Specify the required upstream product and promotion data, placement-based copy quantities, approved ranking data, print-round correlation, and full ticket-validation request/response information. Agree actual schemas, configuration values, and ownership before implementation.

**ER-24:** The handoff identifies each required source and data element, round-result association, and ticket-validation responsibility. Illustrative values are distinguished from the configuration and contracts to be agreed with system owners.

## Traceability summary

| Functional requirements | Business rules | Workflow diagram |
|---|---|---|
| FR-01–03, FR-17 | BR-01–10 | Generation and daily documents |
| FR-04–06 | BR-11, BR-13, BR-25 | Generation and daily documents |
| FR-07–14 | BR-12, BR-14–19 | Printing and recovery |
| FR-15–16 | BR-23 | Reprinting |
| FR-18 | BR-24 | Version changes during printing |
| FR-19–23 | BR-20–22 | Printing and recovery |
| FR-24 | Data and integration dependencies | All diagrams |
