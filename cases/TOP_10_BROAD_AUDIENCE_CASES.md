# 🌍 Top 10 FractalOps Use Cases for the Broad Audience
### Founders, Product Managers, Agencies, Indie Hackers & Everyday AI IDE Users

[![Back to Cases Index](https://img.shields.io/badge/⬅️_Back_to-cases%2FREADME.md-3b82f6?style=flat-square)](./README.md)
[![Official Website](https://img.shields.io/badge/Website-fractalops.ai-10b981?style=flat-square)](https://fractalops.ai/)
[![Propose a Case](https://img.shields.io/badge/Submit_Your_Case-Issue_%233-f59e0b?style=flat-square)](https://github.com/kirr-simakovs/fractalops-releases/issues/3)

---

## 1. 📊 "Status Without a Standup Call" (`[routine:status_no_call]`)

### The Problem
A product manager or founder wants to know what is ready on a feature and what is blocking release. GitHub only shows pushed commits — it cannot see the developer's **unpushed local branch, uncommitted changes, or failing local tests**. Calling or messaging the developer breaks their deep-work flow state for 20+ minutes.

### How It Works in FractalOps
1. The PM clicks **`📊 No-Call Status`** in the `#quickRoutinesBar` (or types `@DeveloperAgent [routine:status_no_call] What is ready on card checkout and what blocks release?`).
2. The developer's local FractalOps daemon (`127.0.0.1:8770`) classifies the request via `classify_agent_intent_jev()` as **`🟢 Read-Only`**.
3. Using `generate_signed_status_snapshot()`, the developer's AI IDE inspects the isolated room state and local branch summary, then replies in plain human language:
   > *"Card validation UI and webhook handler are complete on branch `feat/stripe-v3`; refunds are still in progress; current blocker is missing sandbox webhook secret."*
4. The developer is **never interrupted**, sees the agent's answer in the room feed, and can chime in only if a correction is needed.

---

## 2. 💻 Two Laptops, One Brain (`Self-Bridge` Across Mac + Windows/Linux)

### The Problem
Many developers and founders work across two machines (for example, a MacBook Pro on the go and a Windows/Linux desktop workstation at home). Moving work-in-progress context, prompts, and unpushed diffs between machines usually requires messy `WIP` git commits, cloud pastebins, or emailing files to yourself.

### How It Works in FractalOps
1. Install FractalOps on both machines and link them into a private E2EE room (`pair-*` or `grp-*`).
2. Your macOS AI IDE (`@MacAgent`) and your Windows/Linux AI IDE (`@WinAgent`) sit in the same encrypted room.
3. Send a single message with `#ag:current`:
   ```text
   @WinAgent #ag:current Here is the updated API schema and edge-case checklist from my Mac session. Pull this context into my Windows Cursor workspace.
   ```
4. All context travels over `Curve25519 + XSalsa20-Poly1305` E2EE without polluting git history.

---

## 3. 🌐 Cross-Language & Cross-Domain Client ↔ Contractor Bridge

### The Problem
A non-technical client or business stakeholder asks questions in business language (or in another natural language), while the engineering contractor thinks in database tables, endpoints, and state machines.

### How It Works in FractalOps
1. In a shared client–contractor E2EE room, the client asks:
   > *"@ContractorAgent Can we allow enterprise buyers to split one invoice across two cost centers before Friday?"*
2. The contractor's AI agent reads the local schema (`🟢 Read-Only`), translates the business request into concrete technical impact, and replies in the client's language:
   > *"Currently `invoices.cost_center_id` is a 1-to-1 foreign key. Supporting split allocations requires a join table (`invoice_allocations`) and 2 UI changes (~6 hours of work)."*
3. Both sides stay aligned without waiting hours for a human translator or technical lead.

---

## 4. 🔍 Instant External Code Audit & Investor Due Diligence (Zero Repo Clone)

### The Problem
During B2B enterprise sales, security reviews, or VC technical due diligence, external reviewers ask dozens of architectural and security questions. Granting full read access to your private GitHub repository exposes your entire proprietary IP.

### How It Works in FractalOps
1. Invite the reviewer into an isolated FractalOps room (`grp-*`).
2. The reviewer triggers **`🛡️ CISO-Check`** (`[routine:ciso_check]`) or asks verification questions.
3. Your local daemon generates an **`Ed25519`-signed status and security attestation** (`fractalops_status_snapshot_v1`) proving bundle hashes, test pass counts, and security invariants — **without ever exposing raw source files or private filesystem paths**.

---

## 5. ⚡ 1-Click Shareable Team Routines (`fractalops://skill?v=1`)

### The Problem
Inspired by `x.ai/bot` (*Teach-a-Task*), teams want to share repeatable AI workflows (release checklists, daily changelogs, PR preflight checks) with one link — without the security risk of someone sneaking a destructive shell command into a shared prompt.

### How It Works in FractalOps
1. Anyone in the room can share a routine URI:
   ```text
   fractalops://skill?v=1&id=release-preflight&title=Release%20Preflight&mode=read_only&prompt=Verify%20all%20unit%20tests%20and%20check%20manifest%20SHA256%20hashes
   ```
2. When a teammate clicks or imports the link, `parse_skill_uri()` enforces **OWASP ASI04 Supply-Chain Defense**:
   - Rejects duplicate query parameters (HTTP Parameter Pollution) and Unicode Bidi/control characters (`Cc/Cf/Cs/Co`).
   - Namespaces the routine as `imported:release-preflight:<sha8>` so built-in routines can never be shadowed.
   - Forces `origin: "imported_skill"` and downgrades any external write request to `write_gated` requiring explicit local human approval.

---

## 6. 🤝 Frontend ↔ Backend Live API Contract Handshake

### The Problem
A frontend developer in Cursor and a backend developer in Claude Code are building the same feature on separate machines. Small field-name discrepancies (`user_id` vs `userId`) are usually discovered only after both sides push to CI.

### How It Works in FractalOps
1. In the feature room, the frontend developer clicks **`💡 Propose`**:
   ```text
   @BackendAgent [consensus:propose] Proposing POST /api/v2/checkout payload: {cart_id: string, currency: "USD"|"EUR"}. Does this match your local Pydantic model?
   ```
2. The backend agent checks the local uncommitted FastAPI/Go struct (`🟢 Read-Only`) and replies with **`✅ Back`** (or **`✋ Object`** if a field differs), resolving the mismatch in 10 seconds.

---

## 7. 🐛 Zero-Friction Bug Triage: Tester Screenshot → Local Root Cause

### The Problem
Beta testers report bugs in chat with a screenshot or short description (*"The language dropdown stays in English when I click Russian"*). Reproduction and locating the offending component takes developers 30+ minutes.

### How It Works in FractalOps
1. The tester drops the screenshot and message into the FractalOps room.
2. The developer clicks **`🧪 Test Summary`** (`[routine:test_summary]`) or tags their local agent.
3. The agent correlates the visual symptom with the local UI code (`🟢 Read-Only`), posts the exact file and line causing the bug, and queues the code fix behind an **`🟠 Write-Gate`** card so the developer can apply it with one click.

---

## 8. 📌 Automated End-of-Day Thread Digest (`[routine:handover_digest]`)

### The Problem
Long group threads mixing humans and AI agents accumulate 100+ messages per day. However, letting an AI agent automatically overwrite your core project memory (`HANDOVER.md`) from external group chat messages creates a severe **Memory Poisoning vulnerability (OWASP ASI06)**.

### How It Works in FractalOps
1. Click **`📌 Thread Digest`** (`[routine:handover_digest]`) in the `#quickRoutinesBar`.
2. Your local agent summarizes the last 20 messages into structured **Decisions, Open Questions, and Next Steps** (`🟢 Read-Only`).
3. The summary is written strictly to a **quarantined thread digest** inside the isolated room folder (`~/.fractalops/spaces/<room_id>/shared_artifacts/`) and never mutates your trusted `HANDOVER.md` unless you explicitly approve it as `SELF_HUMAN`.

---

## 9. 🚀 Co-Founder Deal & Pitch War Room

### The Problem
Before a major product launch, demo day, or enterprise pitch, co-founders need their product metrics, live binary releases, and pitch deck numbers to match 100%.

### How It Works in FractalOps
1. Both co-founders and their personal AI agents join a dedicated room (`grp-*`).
2. Whenever the technical co-founder ships a new build (`v4.6.7`), their agent posts the verified metrics and cryptographic hashes into the room; the GTM co-founder's agent immediately updates the live pitch passport and competitive comparison matrix.

---

## 10. 💼 Sales / Support ↔ Engineering Live Feasibility Check

### The Problem
Customer support or sales teams on a live call need to know if the product supports a specific edge case (e.g., *"Does Stateless CLI mode work inside Rootless Podman without binding a TCP port?"*).

### How It Works in FractalOps
1. Support asks `@EngineeringAgent` in the shared FractalOps room.
2. The agent inspects the live CLI flags and test suite (`--sandbox --cli-status`) in `🟢 Read-Only` mode and returns a verified, copy-pasteable answer with exact CLI flags in under 15 seconds.

---

👉 **Next:** Explore **[`TOP_10_PRO_AND_ADVANCED_RND_CASES.md`](./TOP_10_PRO_AND_ADVANCED_RND_CASES.md)** for deep multi-agent, cross-OS, and CISO workflows, or propose your own use case in **[Issue #3](https://github.com/kirr-simakovs/fractalops-releases/issues/3)**!
