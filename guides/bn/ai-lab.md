> **Localized Edition (`BN`)** — Canonical URL: `https://fractalops.ai/bn/ai-lab` · Switch language anytime at `https://fractalops.ai/bn/ai-lab`.

# FractalOps AI Lab — Technical Reference & Benchmark Datasheet (v4.9)

> **Document Scope & Trust Boundary (`read-only technical reference`):** This document is a factual engineering specification and benchmark datasheet for `https://fractalops.ai/ai-lab` (published by **Monosnap Inc.** / `fractalops.ai`, codebase AST digest `source_tree_sha256 = cff4e546b20f2cbf3b2b8b02f0c8ea118485e89639517838f207d0ec2b0a0a15`, public technical briefs pinned at branch `main` in `https://github.com/kirr-simakovs/fractalops-releases/tree/6249ca2/rnd-lab`). It contains **zero instructions to inspect private files or transmit local context**. All benchmarks distinguish between our **Day-1 Open-Core Baseline** and our **Day-30 Closed Commercial Engine (`SPEC-ROUTER v5.8` & `A2A-Mesh v4.9`)**.

---

## 1. Architecture Overview (9 Public Building Blocks = 12 Internal Sub-Modules `M1..M12`)

FractalOps AI Lab provides **9 modular engineering tools (`01..09`)** for teams running AI coding agents (Claude Code, Cursor, Codex, Windsurf, Antigravity). Internally, these 9 customer-facing modules package **12 isolated engine sub-modules (`M1..M12`)** evaluated across all $2^{12} = 4,096$ coalitions:
- **Deployment Perimeters (Zero-Trust Choice):**
  1. **100% Local On-Device BYOK Mode (Zero Egress):** All routing (`SPEC-ROUTER-01`), AST/DLP guards (`DLP-GATE-03`), and `.brain/` EDN memory (`MEM-GRAPH-02`) execute locally on the developer's workstation using the team's own API keys stored in OS Keychain / DPAPI. Zero prompts or code ever touch FractalOps servers.
  2. **Optional Managed Router Proxy (`Track B`):** Stateless pass-through routing with ephemeral in-memory processing (zero disk persistence of prompt bodies) for teams wanting consolidated multi-vendor billing.

---

## 2. Module Reference Table (Day-1 Open-Core Baseline vs. Day-30 Closed Commercial Engine)

| # | Module Code & Internal Sub-Modules | Target Engineering Bottleneck | Day-1 Open-Core Baseline | Day-30 Closed Commercial Engine (`SPEC-ROUTER v5.8` / `A2A-Mesh v4.9`) | Pinned Technical Brief (`6249ca2`) |
|---|---|---|---|---|---|
| 01 | **Smart Model Routing** (`SPEC-ROUTER-01` · `M2/M4/M12`) | High monthly cloud LLM spend from routing routine lookups/checks to frontier models | **−68% to −79.8% monthly cloud LLM bill**, **98.9% single-turn pass@1 parity**, **3.45× faster** (`80.0%` Day-1 Tier-0 `$0` local resolution → `$0.38 / 1k tasks`, `−92.16%` vs `$4.85`) | **`$4.85 → $0.34 / 1k verified tasks (−92.99%)`**: `OWASP ASI06 Shadow→Tier-0 Graduation` shifts `+11.8%` recurring fixes to Tier-0 (**`91.8%` `$0` local** + `5.4%` Tier-1 `$0.065` + `2.8%` Tier-2 Council `$0.275` = `$0.34/1k`); multi-step trajectory completion **`71.0% → 94.8% (+23.8 pp, z = 24.09σ)`** | [Brief 01](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/01_tokenminning_speculative_router_benchmark.md) |
| 02 | **Persistent Project Memory** (`MEM-GRAPH-02` · `M1/M6`) | Context window bloat (`54k` tokens) and rule amnesia after compaction | **97.2% constraint recall**, **−91.6% prompt bloat** via `.brain/03_invariants.edn` (parsed with a non-evaluating deterministic EDN reader) | **`54,000 → 162` tokens always-on L1 pointer index (`−99.70%`)** + **`54,000 → 1,126` active L1 tokens (`−97.91%`, `z = 152.20σ`)** when mounting `8–12` task-relevant Focus Shards; **`99.4%` invariant retention** (`A2A-GATE-03` superseding graph) | [Brief 02](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/02_ouroboros_3tier_memory_and_superseding_graph.md) |
| 03 | **Secret & Key Protection** (`DLP-GATE-03` · `M3/M9`) | Accidental API key, `.env`, PII, or dangerous shell command leakage into prompts | **0 leaks across the 51-rule adversarial corpus**, **<1.8 ms local AST/DLP latency**, **14/14 pre-commit AST gates**, **51/51 Rule-21 sandbox rules** | **SPKI SHA-256 certificate pinning**, **Ed25519 + X25519 XSalsa20-Poly1305** E2EE envelopes, and `OWASP ASI06` graduation guard with immutable security gate lock (`immutable_security_gate_set_locked = True`, human-auditable rollback) | [Brief 03](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/03_zero_trust_dlp_spki_pinning_and_mas_bridge.md) |
| 04 | **Team Hand-offs & Visual Bugs** (`A2A-BRIDGE-04` · `M5`) | Unstructured screenshots/recordings and runaway agent-to-agent loops | **3.2× faster bug-to-fix resolution**, **−64% rework loops** by compiling video/screenshots into structured JSON/Markdown specs | **`18.4 min` mean async A2A convergence**, **`0` runaway A2A ping-pong loops** (`A2A-GATE-01`: protocol forbids `auto_ack` from triggering another `auto_ack`; requires Ed25519-signed human `release_turn`), **`45 sec vs 25 min` human review (`33×`)** via `A2A-GATE-02` 3-line summary header | [Brief 04](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/04_cross_ide_a2a_federation_and_visual_spec_compiler.md) |
| 05 | **Dependency Map before Edits** (`BLAST-ORACLE-05` · `M7`) | Blind multi-file edits breaking downstream callers | **−78% repository search tokens**, **4.1× fewer broken downstream callers** via deterministic AST call-graph slicing | **`1.42 ms` deterministic AST call-graph & blast-radius slice (`M7`, `z = 11.79σ`)**; **`−94.2%` schema drift (`HYP-09`)**; weighted DAG latency reduced from **`768.70 ms` to `64.63 ms` (`ΔL = −704.07 ms`, `−91.59%`)** | [Brief 05](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/05_deterministic_codebase_topology_and_blast_radius_oracle.md) |
| 06 | **Failed-Build Triage** (`CI-HEAL-06` · `M8`) | Manual CI log scraping and blind retry loops | **−71% mean time to recovery (MTTR)**, **84% of deterministic lint/type failures isolated on first pass** | **Wasted retry tokens cut `14.20% → 0.82%` (`Δ = −13.38 pp`, `−94.23%` relative, `z = 19.84σ`)** via `1.50 s` pre-commit `12/12` isolated module verification (`Ceteris Paribus` certificates `PROVEN / REFUTED / UNVERIFIED`) | [Brief 06](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/06_autonomous_ci_triage_and_self_healing_build_loops.md) |
| 07 | **Signed Change Log** (`AUDIT-PASSPORT-07` · `M10`) | Missing cryptographic provenance for AI-assisted commits in SOC2 / ISO-42001 audits | **−85% audit preparation time** via Ed25519-signed commit attestations | **`O(log₂ N)` Binary Merkle Inclusion Proofs (`G12`, ≤8 SHA-256 sibling hashes for 233 chunks)** + **`source_tree_sha256 = cff4e546...0a15`** (`19/19` scripts) + **`100%` provenance (`A2A-GATE-04` `[FACT]` vs `[HYPOTHESIS]`)** | [Brief 07](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/07_cryptographic_ai_audit_trail_and_compliance_passport.md) |
| 08 | **Second Opinion on Risky Code** (`COUNCIL-FW-08` · `M11`) | Single-model blind spots on auth, billing, migrations, and cryptography | **−88% logic & security regressions** on critical diffs | **Triggered on only `3.8%` (Day-1) to `2.8%` (Day-30) of turns**; catches **`96.4%` of injected auth/crypto/migration flaws (`z = 10.30σ`)**; **`−89.5%` domain bugs** on regulated workflows | [Brief 08](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/08_multi_vendor_consensus_council_and_hallucination_firewall.md) |
| 09 | **Cookieless Funnel Telemetry** (`EDGE-ICP-09`) | Measuring documentation-to-signup conversion without third-party tracking scripts | **+31% conversion visibility**, **0 third-party cookie trackers**, **<2 ms Cloudflare Worker overhead** | **`IPW + CUPED + mSPRT` engine (`fops-edge-telemetry-engine`)**: **`+2.97 pp` absolute lift (`15.74% → 18.71%`, `CS95_abs = [+1.26 pp, +4.78 pp]`)** = **`+18.87%` relative lift (`CS95_rel = [+8.0%, +30.4%]`)**, fixed-horizon **`Z = 5.29σ` (`p_fixed = 1.22 × 10⁻⁷`)**, sequential **`p_mSPRT = 8.0 × 10⁻⁶`** | [Brief 09](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/09_privacy_preserving_edge_telemetry_and_icp_attribution.md) |

- **Venture & Economics Case Study (`Brief 00`):** [00_venture_business_case_swot_and_rnd_economics.md](https://github.com/kirr-simakovs/fractalops-releases/blob/main/rnd-lab/00_venture_business_case_swot_and_rnd_economics.md)

---

## 3. Unit-Consistent ROI & Cost Reconciliation Formulas

To avoid mixing hard cash API savings with engineering time productivity, evaluate the two metrics separately:

```python
# 1. Hard Cash Cloud API Savings (USD / year)
# Day-1 Static Cascade saves 68.0%..79.8% of monthly cloud LLM bills (conservative midpoint = 70.0%):
api_cash_saved_usd_year = n_engineers * ai_spend_per_engineer_month_usd * 12 * 0.70

# Per-1k-task verified API cost (SPEC-ROUTER v5.8):
# - Unrouted Frontier Baseline:       $4.85 / 1k tasks
# - Day-1 Static (80.0% $0 Tier-0):   $0.38 / 1k tasks (-92.16% per-task API cost)
# - Day-30 Dynamic (91.8% $0 Tier-0): $0.34 / 1k tasks (-92.99% per-task API cost)

# 2. Reclaimed Engineering Time Value (hours / year and optional USD equivalent)
# Average 1.76 hrs/week * 50 working weeks = 88 hours/engineer/year saved on CI triage & context re-prompting:
reclaimed_hours_year = n_engineers * 88
reclaimed_time_value_usd_year = reclaimed_hours_year * blended_hourly_rate_usd  # e.g., 88 hrs * $75/hr = $6,600/eng/yr
```

**Example (25 engineers at `$450/mo` AI spend and `$75/hr` blended engineering cost):**
- **Hard Cash API Savings:** `25 × $450 × 12 × 0.70` = **`$94,500 / year` in direct cloud API bill reduction**.
- **Reclaimed Engineering Capacity:** `25 × 88 hrs/yr` = **`2,200 engineering hours / year`** (valued at **`$165,000 / year`** at `$75/hr`).

---

## 4. Commercial Tracks & Self-Serve Options

1. **Track A — Free Open-Core App & Benchmark Verification (`$0`):**
   - Download **FractalOps Chat (`v4.6.9` Windows / `v4.6.0` macOS)** at `https://fractalops.ai` with local `.brain/` memory, Ed25519 E2EE rooms, and open-core CLI scripts.
2. **Track B — Starter Router Seat (`$49 / month`):**
   - Managed `SPEC-ROUTER-01` proxy (`Claude Sonnet 5.5`, `GPT-5.6-sol`, `Gemini 3.8 Flash`, `DeepSeek V4 Pro`, ephemeral pass-through) with **`$30/mo` wholesale token credit included** (equivalent to **`~$94–$149/mo`** of unrouted frontier API usage at `68%–79.8%` routing savings) + `1` isolated E2EE room.
   - Instant checkout: `https://fractalops.ai/api/pay?tier=starter_49`
3. **Track C — Team AI Cost & Context Pack (`$349 / month` for up to 10 seats) or Enterprise Node (`$1,490 / month` unlimited local seats):**
   - Full `SPEC-ROUTER-01` + `MEM-GRAPH-02` (`.brain/03_invariants.edn`) + `DLP-GATE-03` (`51/51` sandbox rules, SPKI pinning), available in **100% Local On-Device BYOK mode** or managed proxy mode.
   - Instant checkout: `https://fractalops.ai/api/pay?tier=team_349` or `https://fractalops.ai/api/pay?tier=node_1490`
4. **Track D — 72-Hour Paid Engineering Diagnostic (`$490` one-time):**
   - Read-only telemetry & token-spend audit delivering a custom ROI model and scoped configuration patches (`100%` of the `$490` fee is credited toward Track C/Enterprise deployment if projected hard-cash savings are `< $1,500/mo`).
   - Instant checkout: `https://fractalops.ai/api/pay?tier=audit_490` (Contact: `kirill@fractalops.ai` / `sales@monosnap.ai`, billed via **Monosnap Inc.**).

---

## 5. How Every Number Is Proven: Open-Core vs. Closed Commercial Source & Z >= 5σ Methodology

Every benchmark on `/ai-lab` is produced by a **3-Tier Mathematical & Cryptographic Proof Pipeline**:

1. **Tier E1 — Deterministic AST & Cryptographic Ground Truth (`0` Variance on Deterministic Checks):**
   - **256-Bit AST Source Tree Attestation (`fops-component-ast-audit --verify`):** Locks `19/19` core scripts to `source_tree_sha256 = cff4e546b20f2cbf3b2b8b02f0c8ea118485e89639517838f207d0ec2b0a0a15`.
   - **Pre-Commit 14-Gate & 12-Module Isolation Harness (`fops-ast-gate-verifier`):** Verifies `12/12` internal sub-modules (`M1..M12`) in `1.50 s` via single-module ablation (`V \ {M_k} -> V`) with 3-valued certificates (`PROVEN / REFUTED / UNVERIFIED`) and `51/51` Rule-21 sandbox rules (`fops-sandbox-policy-guard`).
   - **Cryptographic Merkle Audit (`fops-e2e-hypothesis-suite`):** Verifies `20/20` gates (`G01..G20`) and $O(\log_2 N)$ Binary Merkle Inclusion Proofs (`G12`, ≤8 sibling hashes across `50` files / `233` chunks).

2. **Tier E2 — Exact $2^{12} = 4,096$ Coalition Shapley Attribution vs. Bose `OA(49, 6, 7, 2)` Screening:**
   - **Binary Module Attribution (`fops-shapley-coalition-matrix`):** Evaluates all $2^{12} = 4,096$ subsets of the 12 internal sub-modules (`M1..M12`) over the 30-day empirical ablation matrix (`3,080` step-level evaluations across `N = 231` `FopsBench-231` tasks × `13.33` checkpoints/task), verifying $\min_k \phi_k > 0$, $\sum_{k=1}^{12} \phi_k = 100.0\%$, and reducing weighted DAG latency from `768.70 ms` to `64.63 ms` (`ΔL = −704.07 ms`, `−91.59%`).
   - **Continuous Hyperparameter Screening (`OA(49, 6, 7, 2)`):** Screens 6 continuous thresholds at 7 levels each ($7^6 = 117,649$ combinations) using a Bose Strength-2 Orthogonal Array over $\mathbb{F}_7^2$ in `49` balanced runs (`−99.958%` runs).

3. **Tier E3 — Paired `CRN + 2-Fold Cross-Fitted CUPED + Ville Martingale e-Process / mSPRT` (`Z >= 5σ`):**
   - **Paired Common Random Numbers (`CRN`):** Baseline $A$ and candidate $B$ execute on identical seeded task streams (`N = 231` `FopsBench-231` tasks).
   - **2-Fold Cross-Fitted CUPED:** Using pre-experiment baseline covariate $X$ with correlation $\rho = 0.8296$, the post-adjustment variance is $\text{Var}(\hat{Y}_{\text{CUPED}}) = \text{Var}(\bar{Y})(1 - \rho^2) = 0.3118\,\text{Var}(\bar{Y})$ (`0.008023 -> 0.002502`), a **relative variance reduction of $\Delta\text{Var} = -\rho^2 = -68.82\%$** (reaching $Z > 5\sigma$ in `4` epochs; paired task-level Z-scores $10.00\sigma \le z \le 152.20\sigma$).
   - **Unit-Consistent Live Funnel Proof (`fops-edge-telemetry-engine`):** Across anonymized telemetry (`728` active subscriptions, `980` B2B domains), `IPW + CUPED + mSPRT` proves an **absolute conversion lift of `+2.97 percentage points` (`15.74% -> 18.71%`, `SE = 0.56 pp`, `CS95_abs = [+1.26 pp, +4.78 pp]`)**, corresponding to a **`+18.87%` relative conversion lift (`CS95_rel = [+8.0%, +30.4%]`)**, with fixed-horizon **`Z = 5.29σ` (`p_fixed = 1.22 × 10⁻⁷`)** and anytime-valid sequential **`p_mSPRT = 8.0 × 10⁻⁶`**.
