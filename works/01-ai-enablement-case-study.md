---
---

# From Manual Data Verification to an Automated Deviation Report
*Engineering data maintenance before code freeze — case anonymized*

> **Confidentiality & anonymization notice:** this task involves customer-sensitive engineering data. Platform names, file formats, and data-handling specifics are **not disclosed**. What is shown in full is the transferable part: the problem analysis, the comparison logic, the workflow, and the results. Everything described is real; nothing is invented.

**TL;DR** — Manual verification of structured data files before a code freeze was error-prone and squeezed into a 2-hour window. I built a small tool (Python and PowerShell) — developed with Copilot in Visual Studio Code as a coding assistant — that compares the target file against the stakeholder's input files (.txt / Excel), applies the domain rules, and returns a **deviation report**. The reviewer reads a short report instead of re-checking every value by eye. **Result: 83% reduction in working time and zero errors in the following 4 cycles, standardized in an SOP and handed over to colleagues.**

## 1. Context
Engineering data for OEM projects is maintained in a central platform as structured files. Preparing a change before code freeze follows a fixed cycle:
1. receive a stakeholder file with new values
2. download the required labels from the platform (as a data file)
3. update the label values
4. verify the result before upload

Step 4 — verification — is where hidden errors lived.

## 2. The problem — three failure vectors
- **Manual and error-prone**: values were checked by eye; a slip often stayed invisible until the next release, when a data deviation finally surfaced.
- **Extreme time pressure**: the window before freeze was sometimes only 2 hours — preparing *and verifying* the whole set inside it was critical and stressful.
- **Hidden domain complexity**: several vehicle IDs per file, each with several labels; each label carries two values, only one of which is authoritative (the parallel value must be ignored); the stakeholder file uses German comma decimals while the platform file uses dots; only labels present in the stakeholder file may change; labels missing from the platform must be reported, never invented.

## 3. The approach — automate the comparison, keep the judgement
Rather than verifying by eye, I specified the rules and built a small tool (Python and PowerShell) that:
- reads the target file and the input file(s) (.txt / Excel)
- applies the domain rules — authoritative value only, comma → dot, missing labels flagged rather than guessed
- emits a **deviation report**: a short table of exactly what changed and what did not, with an explicit audit line for anything left untouched

The tool reports; the human decides. Nothing is silently "fixed" — deviations are surfaced, not covered up.

## 4. How it was built — and where the AI sits
I developed the tool with **Copilot in Visual Studio Code** as a coding assistant, and refined it over multiple iterations, each driven by the results it produced in real use — the tool was treated as a product, not a one-off script.

Crucially, the AI sits on the **code** side, not the **data** side: everything ran locally on my own machine — the data files, the Excel inputs, and the generated deviation reports never left it. The AI assistant helped write and iterate the code; the comparison itself is a deterministic local script. **AI for building, not for handling the data** — a deliberate design rule, not an accident.

## 5. Results
- 83% reduction in working time for this task
- Zero errors in the following 4 cycles
- The task stopped being a bottleneck in the tight maintenance window
- Process standardized in an SOP; colleagues coached to run the workflow independently

## 6. Beyond the tool
The tool was the easy part. The durable value: an SOP, trained colleagues, and a check that no longer depends on one person. **The capability outlives the builder.**

## 7. The transferable method (AI-enablement playbook)
1. **Find the repetitive verification bottleneck** — the step that repeats, eats time, and carries the highest error cost
2. **Automate the check, not the judgement** — produce a report; let a human decide
3. **Encode the domain rules explicitly** — the tool doesn't know your domain (decimal conventions, which value is authoritative, missing-data policy); you encode it
4. **Keep AI out of the data path** — use AI to write and iterate the code; keep the data on the local machine
5. **Standardize and enable** — SOP + coaching, so the capability outlives the builder

## 8. Limitations & next steps
- No formal error-rate measurement at scale
- The rule set is specific to this data model — a new file format needs new rules
- Handover still assumes a user who can run a script; a no-code wrapper is a possible next step
