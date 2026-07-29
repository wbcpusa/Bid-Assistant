# Handover — CP USA "Bid Assistant" Tuning Branch
**Purpose:** Give a fresh Claude session (Claude Code, primary) everything it needs to help refine and fine-tune CP USA's Bid Assistant. This document is the context bridge from the strategy/build chat where the tool originated.
**Prepared:** July 2026 · **Owner:** Will Burdick, Dallas Branch Manager / AI Specialist, Control Panels USA (CP USA)

---

## 1. Who CP USA is (context for judging scope)
CP USA is an instrumentation & controls (I&C) contractor and UL 508A control panel manufacturer. On a typical project we furnish and/or provide some combination of: control panels (PLC panels, RTU panels, starter/VFD panels, relay racks, OEM enclosures), PLC/RTU hardware and programming, HMI/SCADA/OIT configuration, field instruments (supply, calibration, loop checks), industrial networking (managed switches, routers, firewalls, network management), fiber optic cable/terminations/testing, radio/telemetry, control-panel UPS units, and the associated testing, startup, commissioning, training, and O&M documentation. We often act as the PCSI (systems integrator) and sometimes the ASP (application services provider / programming) as well.

We work heavily in municipal water/wastewater (e.g., Trinity River Authority) and industrial markets. Specs are usually CSI/MasterFormat (Division 40 = our home base; also Div 26, 27, 43/44/46 vendor packages, and Div 00/01 front-end). Some specs use older or custom numbering — the tool must detect the format, not assume it.

## 2. What we're building — the "Bid Assistant"
**One combined tool that reads a project's specifications AND contract drawings together** and produces a predicted CP USA scope: the equipment and services we'd propose, called-out manufacturers/part numbers, commercial risks and buried requirements, likely change-order exposure, and suggested bid-time questions/clarifications. Specs and drawings are read together on purpose — neither alone gives the full picture (spec says "per drawings," drawings show counts the spec omits, etc.). This is NOT two tools; it is one Bid Assistant.

It has an existing predecessor — a spec-only "Spec Scanner" instruction prompt (v1.1), already field-tested. **That prompt is provided as a separate file in the project folder** (do not expect it inline here). The tuning branch's job is to evolve that prompt into the combined, sharper Bid Assistant.

## 3. The goal of THIS branch (what Claude Code should help do)
Refine and fine-tune the Bid Assistant's instruction prompt by working through past CP USA projects where we have the ground truth (the proposal we actually bid). Concretely:

1. Read each past project's inputs: **specifications + contract drawings + the proposal CP USA actually submitted.**
2. Run the current Bid Assistant prompt against the spec+drawings, producing a predicted scope/report.
3. Compare prediction vs. the actual proposal. Log every discrepancy.
4. Use what's learned to propose targeted improvements to the instruction prompt.
5. Repeat across many projects so the tool learns CP USA's *standard* scope.

**This is interactive, not one autonomous swoop.** Set up understanding first; then Will works with it project by project. It may run through multiple projects on its own when asked, but the default is collaborative refinement.

## 4. The single most important principle — do not overfit to one job
A proposal reflects what CP USA *bid on that specific job*, which is not always our standard scope. We sometimes exclude something we normally provide because we negotiated it with the customer, value-engineered it, or the job simply didn't need it. **A "in the spec but not in our proposal" delta is therefore NOT automatically a scanner miss.** It might be a correct, deliberate business decision.

Rule for the tool and the journal: treat proposal-vs-prediction deltas as **flags for human review**, never as automatic corrections. Will tells it which deltas are real misses (fix the prompt) vs. job-specific exceptions (record the reason, do NOT generalize). If the tool "learns" that we don't provide something just because one proposal omitted it, it will under-report on future bids — the exact failure to avoid.

## 5. The journal (persistent learning log)
Maintain a running journal/log across all reviews — this is the institutional-memory layer, as important as the prompt itself. For each project reviewed, log:
- Project ID / customer / job type.
- What the Bid Assistant predicted (scope, equipment, services, part numbers).
- What CP USA actually proposed.
- Every discrepancy, classified: **(a) real miss** — tool should have caught it, prompt needs fixing; **(b) job-specific exception** — we deliberately didn't provide it here, with the reason; **(c) tool over-reach** — tool predicted something out of our scope.
- Items the tool flagged that turned out valuable (catches), and the reason.
- Any new landmine category, customer-specific rule, or manufacturer/vocabulary gap discovered.
- Proposed prompt change (if any) and rationale.

Keep it as a durable file (e.g., `journal.md` or per-project entries) so future sessions can read the history and build on it. The refined prompt distills what the journal has learned; the journal is never discarded.

## 6. Suggested folder structure (for the local corpus)
```
/BidAssistant/
  instruction_prompt.md        # current prompt (evolving)
  journal.md                   # persistent learning log
  /projects/
    /22-4332_TRA_FilterRehab/
      /specs/
      /drawings/
      proposal.pdf             # what we actually bid = ground truth
    /<next project>/
      /specs/  /drawings/  proposal.pdf
    ...
  /output/
    <project>_bid_report.md    # tool's predicted scope per project
```
Will will drop many projects in `/projects/`. The tool reads each project's specs+drawings, writes a predicted report to `/output/`, compares to that project's `proposal.pdf`, and updates `journal.md`.

## 7. What the Bid Assistant should ultimately produce (per bid)
A section-cited report covering: CP USA scope summary (equipment + services), called-out manufacturers/part numbers (marking sole-source "no substitutions" vs. approved-equal vs. "or equal"), commercial/procurement flags (owner-approved-vendor requirements, warranty duration + start triggers incl. delayed-start clauses, testing gauntlet with notice periods, spares, licensing, owner standards "available on request"), interfaces with other divisions (scope-boundary register), predicted change-order exposure (deltas between bid set and any conformed set/addenda), and suggested bid questions/clarifications. Every factual claim carries its spec section or drawing sheet reference. Every report ends with an explicit limitations section (what was fully read vs. keyword-scanned vs. not covered; tables that lost structure in extraction; anything needing native-PDF or image verification).

## 8. Known technical realities (learned the hard way)
- Large spec sets can exceed a single run's tool/operation budget — the current prompt uses a **stage + checkpoint** protocol (index/front-end → I&C deep read → cross-reference sweep → proposal delta → report assembly), saving findings between stages so a limit-hit resumes with "continue" instead of restarting.
- **Tables** (I/O lists, PLC schedules, instrument schedules) frequently lose column structure in text extraction — report their existence confidently, contents only when structure is intact, and parse from native PDF or page images when counts matter.
- Some PDFs are **scanned images** with no text layer — detect this and switch to page-image reading of I&C sections.
- **Drawings** are image-heavy — reading them means vision over sheets: classify sheets touching CP USA scope, extract instrument tags / I/O points from P&IDs, pull equipment schedules into tables. Validate drawing extraction against completed jobs where the as-built I/O list exists.
- Spec numbering format varies — **derive the section-header pattern from the actual document**, never assume MasterFormat.

## 9. Tooling decision (why Claude Code for this branch)
Claude Code was chosen over Cowork for tuning because the work is file-and-filesystem native: point it at the `/BidAssistant/` tree, read across many projects, write the refined prompt and journal as files, loop through the corpus. The refined `instruction_prompt.md` is the portable deliverable — it later drops into whatever the estimators actually use (a Claude Project today, the company Teams "Bid Assistant" later). Code is the workshop; it is not what end users touch.

## 10. Immediate first steps for the Claude Code session
1. Read `instruction_prompt.md` (the current v1.1 prompt) and this handover.
2. Confirm understanding of the goal, the anti-overfitting rule (§4), and the journal schema (§5) back to Will before processing anything.
3. Start with ONE project (suggest 22-4332 TRA Filter Rehab — we have specs, drawings, and the proposal). Run the prompt, produce the report, compare to the proposal, draft the first journal entries, and propose prompt improvements — then review with Will before scaling to more projects.
4. Fold approved improvements into the prompt; bump the version; keep the journal current.

## 11. What is happening in parallel (so this branch stays in its lane)
In the main strategy chat, Will and Claude are continuing the overall AI roadmap, the hosting/IT brief for Eddie & David, the company MCP server / document brain / global Teams assistant design, the Martin update, and the broader 20-idea opportunity portfolio. **This branch should focus only on refining the Bid Assistant tool and its prompt/journal** — not the company-wide strategy. If strategy questions arise, note them for the main chat rather than solving them here.

## 12. Near-term real-world constraint
Will has a meeting with the company president (Martin) to showcase the tool. Demo credibility matters more than feature count: honest limitations stated, section citations on every claim, and real catches on a real project beat an impressive-looking report that can't be trusted. Favor accuracy and defensibility over breadth.
