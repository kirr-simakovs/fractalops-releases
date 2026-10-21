# FractalOps `v4.7.0` — Oct 21 Incident Post-Mortem (`rowid=235`), Follow-Up Security Audit Response (`@bisqet` `rowid=215/216`), Self-Critique Matrix & E2EE Notification Architecture RFC

**Date:** 2026-10-21  
**Authors:** Kirill Simakov (`@Kirill`), Andrey Golovko (`@Andrey`), Dima Tipikin (`@Дима Типикин`) & FractalOps Engineering / CISO Agent  
**External Security Auditors:** David Lukava (`@bisqet`, pubkey `161c92ad6be162ad3c0b76cc372cb61b153951d56ba7cc32382bf3c283c56729`) & `@Агент bisqet` (`FractalOps Security`)  
**Release Version:** `v4.7.0` (`30/30` Zero-Trust Security Tests Passing, `7/7` Space Isolation Checks Passing)

---

## 1. Incident Post-Mortem — Oct 21, 2026 (`18:04` Tbilisi / `14:04 UTC`, `rowid=235` in `grp-b519f703`)

### 1.1 What Happened (Timeline & Observed Behavior)
On **2026-10-21 at 18:04 (Tbilisi)**, David (`@bisqet`) observed an unexpected automated message from `🤖 Агент Kirill` (`a2df50173e32faec730f3b9bce5b886a637766a57cfd179ab5e96ea4bb09a30a`) inside the security audit group room **`grp-b519f703` (`AI-datacenters Security`)**:
```text
@bisqet — E2EE P2P Box (v4.6.9) active.
{"customer_token": "LA4xLG5xLAS5", "status": "relay_blob_delivery_live_v469", "commit": "be90bc7", "relay_box_url": "https://45.77.66.95/box/b7d555807e59", "note": "All 14 security items verified and synced via E2EE Vultr box."}
```
Immediately after (`18:05`), `@Агент bisqet` replied inside `grp-b519f703`:
1. Confirming that the message arrived from pubkey `a2df50173e32faec...`, which was **not** a registered member of `grp-b519f703` on David's `v4.6.5` node (where only Kirill's earlier desktop key `5aec3c82f638be3b2cc099ae2228dab4ab2cf417eecdfac36a72e41d3a3ceef6` was a member), empirically reproducing `AUTHZ-02` on `v4.6.5` clients.
2. Flagging the cross-room leak of the internal workspace marker `"customer_token": "LA4xLG5xLAS5"` and dead probe URL `https://45.77.66.95/box/b7d555807e59` (`HTTP 400 invalid pubkey`).
3. Re-referencing the two detailed follow-up audit messages sent on **2026-10-20 at 04:08–04:10** (`rowid=215` and `rowid=216`).

### 1.2 Root Cause Analysis (3 Intersecting Faults)
1. **Cross-Chat Key Overwrite (`5aec3c82...` $\to$ `a2df5017...`):**
   - On Oct 19, our macOS desktop node joined `@bisqet`'s room `grp-b519f703` using desktop key `5aec3c82f638be3b2cc099ae2228dab4ab2cf417eecdfac36a72e41d3a3ceef6`.
   - On Oct 20 (`ts=1792513838`), a sync script in a parallel IDE workspace (`publish_issue13_mastermind_and_e2ee_sync.py`) rotated `~/Library/Application Support/FractalOps/ed25519.priv` and Keychain `ai.fractalops.ed25519` to Kirill's primary key `a2df50173e32faec730f3b9bce5b886a637766a57cfd179ab5e96ea4bb09a30a` without sending a `hello` / membership migration packet to `grp-b519f703`.
   - Consequently, when `@bisqet`'s node sent replies to `grp-b519f703`, it encrypted them to the room's registered member key `5aec3c82...`, whereas our local daemon was only polling `/box/a2df5017...`!
2. **Unscoped Broadcast Loop in Sync Script (`rowid=218, 227, 231, 235`) + Missing Outbound DLP Gate:**
   - The parallel workspace script iterated over all rooms in `fractalops_v4.db` (including `grp-b519f703`) instead of scoping strictly to `pair-ff411b6c69df` / `grp-1b3d2813`, and `send_room_message()` lacked an outbound Data Loss Prevention (DLP) filter to block internal customer/workspace tokens (`"customer_token": "LA4xLG5xLAS5"`, `bridge.token`) from being sent to third-party rooms.
3. **Single-Room Scope in the 30-Second Poller (`task-738`):**
   - Our 30-second zero-noise poller (`run_zero_noise_poller_7d` in `fractalops_chat_space_manager.py`) was only watching `grp-56813d93` (`AI-datacenters`) rather than all 5 active E2EE rooms (`grp-56813d93`, `grp-b519f703`, `grp-f957da4c`, `grp-1b3d2813`, `pair-ff411b6c69df`), and its localhost trigger `GET /api/rooms` omitted `X-FractalOps-Token` after `v4.6.9` enabled strict token authentication.

---

## 2. Point-by-Point Resolution of `@Агент bisqet`'s Follow-Up Audit (`rowid=215` & `rowid=216`) in `v4.7.0`

| # | Audit Finding (`@Агент bisqet`, `rowid=215` & `rowid=216`) | Root Cause in `v4.6.9` | Exact Fix Implemented in `v4.7.0` | Automated Test |
|---|---|---|---|---|
| **1** | **Manifest Version & Signature Mismatch (`rowid=216` #1):** Live manifest declared `4.6.7` or lacked verifiable `macos` signature block. | `publish_v469_and_respond_all.py` wrote top-level metadata without signing via the dedicated release key `949d690b...` and without nested `macos` parity. | `manifest.json` (`version: "4.7.0"`) now includes both top-level and `macos` blocks, signed with offline release key `949d690b1f9899d04f78cfd33ea3b6537eb5a401fc0326136f4b0b9591e5c0d0` (`FRACTALOPS-RELEASE-SIG-v2\n4.7.0\n<sha256>`) plus `signature_v1_compat` (`a2df5017...`) so `4.6.5` nodes can verify the transition. | `Test #27`, `Test #29` |
| **2** | **`verify_cross_origin_manifest_quorum()` Defined But Not Called (`rowid=216` #2):** `check_and_apply_auto_update()` still took the first manifest URL that responded. | `check_and_apply_auto_update()` broke out of the loop on the first `200 OK` manifest URL. | `check_and_apply_auto_update()` (`p2p_crypto_daemon.py` lines `403-420`) now fetches both `MANIFEST_URL` and `FALLBACK_MANIFEST_URL`, requires `len(fetched_manifests) >= 2`, and calls `verify_cross_origin_manifest_quorum(manifest, secondary_manifest, require_quorum=True)` before downloading any bundle. | `Test #27` |
| **3** | **Local WebSocket `:8999` Unauthenticated & `Origin: ""` / `file://` Allowed (`rowid=216` #3 & `rowid=215` #1):** Any local process could connect to `ws://127.0.0.1:8999` with empty `Origin` and send messages or mutate state. | `ALLOWED_LOCAL_ORIGINS` included `""` and `"file://"`, and `ws_handler` accepted mutation actions without verifying `bridge.token`. | 1) `ALLOWED_LOCAL_ORIGINS = {"http://127.0.0.1:8770", "http://localhost:8770"}` (`""`, `"null"`, and `"file://"` rejected).<br>2) `_ws_auth_ok(websocket)` enforces constant-time `hmac.compare_digest` against `bridge.token` before upgrading.<br>3) `ws_handler` is now **strictly push-only** (`ping`/`status` only; all mutations must use `HTTP POST /api/*` with `X-FractalOps-Token`). | `Test #6`, `Test #21` |
| **4** | **Author Chat Keys in `TRUSTED_UPDATE_PUBKEYS` (`rowid=216` #4 & `rowid=215` #6):** Hot chat keys (`a2df5017...`, `ff411b6c...`) were in `TRUSTED_UPDATE_PUBKEYS` and held special privileges. | Release signing and chat identity shared the same Ed25519 keypair. | `TRUSTED_RELEASE_SIGNING_PUBKEYS` (`p2p_crypto_daemon.py` line `130`) contains **only** dedicated offline release signers (`949d690b...` macOS, `13e61381...` Windows). Hot chat keys (`a2df5017...`, `ff411b6c...`) are completely removed from `TRUSTED_UPDATE_PUBKEYS` and have zero special privileges in chat or IDE bridge. | `Test #29` |
| **5** | **`SELF_AGENT` Provenance, `recommended_skills=[]` for Non-`SELF_HUMAN`, and Double `🤖 [...]:` Prefix (`rowid=215` #3, #4, #5):** Agent self-messages could be treated as self-echo, `recommended_skills` was non-empty for peers, and `send_room_message` doubled `🤖 [Агент Kirill]: 🤖 [Агент Кирилла]:`. | `_build_ide_execution_envelope` returned 3 default skills for peers; `send_room_message` prepended `🤖 [{who_label}]:` without stripping existing `🤖 [...]:`. | 1) Non-`SELF_HUMAN` envelopes now receive `recommended_skills: []`.<br>2) Messages from our own pubkey with `role == "agent"` receive `trust_tier = "SELF_AGENT"` and raise `PermissionError` in `enqueue_ide_bridge_message()` so agent messages **never** re-enter `ide_bridge_inbox.jsonl`.<br>3) `send_room_message()` strips any existing leading `^🤖\s*\[[^\]]+\]:\s*` before formatting wire text. | `Test #28`, `Test #29` |
| **6** | **Outbound DLP Gate (`rowid=235` Incident Fix):** Internal tokens (`"customer_token": "LA4xLG5xLAS5"`, `bridge.token`) could be accidentally sent to external rooms by scripts. | `send_room_message()` had no outbound content inspection for local secrets/tokens. | Added `_enforce_outbound_dlp_gate(room_id, text)` in `send_room_message()` (`p2p_crypto_daemon.py` lines `3868-3892`) blocking `LA4xLG5xLAS5`, `"customer_token"`, `bridge.token`, `sk-fops-*`, and private key markers from leaving to any external room (`room_id != "tech-chat"`). | `Test #30` |

---

## 3. RFC: Efficient Server Notification & Zero-Noise E2EE Polling Architecture

### 3.1 Problem Statement
How should FractalOps nodes and IDE background pollers receive new E2EE messages from the blind relay (`45.77.66.95`) with:
1. **Zero IDE Context Pollution:** Background polling every 30 seconds must emit `0 bytes` to `stdout` when there are no new messages, waking the agent **only** on verified diffs.
2. **Minimal Bandwidth & CPU Overhead:** Full-box polling (`GET /box/<pubkey>`) returning all historical ciphertexts forces the client to re-download and re-parse old blobs on every tick ($O(N)$ network and Curve25519 CPU cost).
3. **Sub-Second Real-Time Wakeup for Desktop & Multi-Agent Swarms:** Interactive chat and A2A collaboration benefit from immediate wakeup without hammering the relay.

### 3.2 Architectural Options Comparison Matrix

| Criterion | Option A: Cursor Delta Polling (`GET /box/<pub>?after_id=<id>` + `If-None-Match` + Local $O(1)$ Pre-Decrypt Skip) | Option B: Authenticated HTTP Long-Polling (`GET /box/<pub>?after_id=<id>&wait=25`) | Option C: Multiplexed Authenticated SSE / WebSocket Push (`GET /stream/<pub>` Wakeup Frame + Delta Fetch) | Option D: Per-Room Blind Sub-Box Routing (`POST /box/<H(room_id \|\| pubkey)>` + Ephemeral Sub-Addresses) |
|---|---|---|---|---|
| **Latency** | `5s` (Desktop UI) / `30s` (Background IDE Daemon) | `< 150 ms` (instant return on new message arrival) | `< 50 ms` (instant push over persistent TLS connection) | Depends on A/B/C transport (`< 100 ms` with B/C) |
| **Bandwidth & CPU Cost** | **$O(\Delta)$ network, $O(\Delta)$ Curve25519 decrypt** (`304 Not Modified` or `[]` when idle) | **$O(\Delta)$ network & CPU**, 1 request per 25s when idle | **Near-zero idle traffic** (1 keepalive frame / 30s) | **$O(\Delta_{\text{room}})$** — isolates high-volume rooms from 1:1 inbox |
| **CISO / Zero-Trust Security** | Reuses exact per-request Ed25519 signature (`X-Signature`, `X-Timestamp`, `X-Nonce`) | Reuses exact per-request Ed25519 signature; bounded connection hold time | Requires short-lived challenge-response token on stream open + re-auth every 15m | Hides social graph & prevents uninvited cross-room spam at relay layer |
| **NAT / Firewall / Proxy Resilience** | **100%** (standard stateless HTTP GET) | **99%** (works through all standard HTTP/1.1 & HTTP/2 proxies) | **90%** (some corporate proxies drop idle WS/SSE without 25s heartbeats) | **100%** |
| **Implementation Complexity** | **Low** (client-side live in `v4.7.0`; 10-line relay filter) | **Low-Medium** (async `asyncio.Event` per pubkey on relay) | **Medium** (connection registry + backpressure on relay) | **Medium-High** (requires capability sub-key rotation on member kick) |

### 3.3 Selected Optimal Architecture (3-Stage Rollout)
1. **Stage 1 — Shipped Today in `v4.7.0` (Client-Side Cursor Delta Polling + Multi-Room Zero-Noise Daemon):**
   - `poll_vultr_once()` now tracks `_LAST_RELAY_CURSOR_BY_PUBKEY[my_pub]` and `_SEEN_BOX_IDS_MEM`, sends `?after_id=<last_cursor>` + `If-None-Match: "<last_cursor>"` + `X-FractalOps-After-Id: <last_cursor>`, honors `HTTP 304 Not Modified`, and skips any already-seen `msg_id` or `blob_sha256` in $O(1)$ **before** invoking Curve25519 `Box.decrypt()`.
   - `run_zero_noise_poller_7d()` (`fractalops_chat_space_manager.py --daemon-poll-30s-7d`) now authenticates to localhost `:8770/api/poll` using `X-FractalOps-Token`, monitors all 5 active rooms (`grp-56813d93`, `grp-b519f703`, `grp-f957da4c`, `grp-1b3d2813`, `pair-ff411b6c69df`), writes heartbeats silently to `RAG/poller_7d_heartbeat.json` (`0 bytes` to `stdout` on no-diff), and emits a single JSON line **only** when a brand-new external message or GitHub issue update arrives.
2. **Stage 2 — Relay Server Upgrade (`v4.7.1` / Joint Relay Patch): Cursor Filtering + `?wait=25` Long-Polling:**
   - Add `?after_id=<id>` slice filtering and optional `?wait=25` `asyncio.Event` wakeup on the FastAPI/Nginx relay at `45.77.66.95` so idle polls drop from 12 req/min to 2.4 req/min with `<150 ms` delivery latency.
3. **Stage 3 — `v4.8.0` Roadmap: Per-Room Blind Sub-Boxes (`H("fractalops-subbox-v1" || room_id || member_pubkey || epoch)`):**
   - Each room member receives messages for `grp-*` in a capability-scoped sub-box. If a member leaves or is kicked, or if an uninvited key sends to `grp-*`, the relay rejects or isolates the traffic before it ever reaches the main identity inbox.

---

## 4. Maximal Self-Critique — All Identified Weaknesses, Candidate Solutions & Selected Optimal Fixes

We performed an adversarial CISO / OWASP ASI01–ASI10 self-audit across the entire FractalOps stack. Below are the **8 remaining architectural weaknesses / blind spots**, the candidate solutions evaluated for each, and the selected engineering resolution:

### Weakness 1: Multi-Process / Multi-Chat Key Contention on `~/Library/Application Support/FractalOps/ed25519.priv`
- **Problem:** Multiple IDE chats or helper scripts running on the same macOS host shared a single mutable `ed25519.priv` / Keychain slot (`ai.fractalops.ed25519`). When one script rotated the active key (`5aec3c82...` $\to$ `a2df5017...`), rooms joined under the previous key (`grp-b519f703`) lost membership alignment (`rowid=235` incident).
- **Candidate Solutions:**
  1. *Option 1A:* Forbid scripts from ever overwriting `ed25519.priv` once initialized; store secondary/historical keys in a read-only `keyring/` directory so `poll_vultr_once()` decrypts incoming boxes for **both** the primary key and any archived local keys.
  2. *Option 1B:* Separate `FRACTALOPS_DATA_DIR` per IDE workspace.
  3. *Option 1C:* Signed key-rotation announcement packet (`{"type": "key_migration", "old_pubkey": ..., "new_pubkey": ...}`) co-signed by both old and new Ed25519 keys.
- **Selected Optimal Solution (`1A + 1C`):** Lock primary key against silent overwrite (`1A`), poll archived local keys for backwards compatibility, and use dual-signed `key_migration` / `hello` packets when migrating room membership (`1C`).

### Weakness 2: Client-Fanout Group Encryption ($O(M)$ Pairwise NaCl Boxes Without Shared Room Epoch Key)
- **Problem:** In `v4.x`, sending a message to a group of $M$ members encrypts $M$ separate pairwise Curve25519 boxes (`::room grp-xxx::` inside the plaintext). While `v4.6.9+` verifies `sender_pubkey` against local `members`, a `v4.6.5` peer that hasn't upgraded (`@bisqet`'s node during `rowid=235`) still routes by the `::room grp-xxx::` prefix (`AUTHZ-02`). Furthermore, pairwise fan-out lacks forward secrecy (compromise of a static Ed25519/Curve25519 key decrypts historical boxes).
- **Candidate Solutions:**
  1. *Option 2A:* Sender Keys (Signal-style per-sender symmetric ratcheting chain key per room `K_{room, sender}^{(i)}` + inner Ed25519 signature over `(room_id, epoch, msg_seq, ciphertext)`).
  2. *Option 2B:* Full RFC 9420 Messaging Layer Security (MLS) TreeKEM for dynamic groups.
  3. *Option 2C:* Keep pairwise `crypto_box` but add a mandatory `room_membership_token` signed by `creator_pubkey` inside every group message header.
- **Selected Optimal Solution (`2C` immediate in `v4.7.x` $\to$ `2A` Sender Keys in `v4.8.0`):** `2C` is 100% backwards-compatible with existing relay boxes while proving room membership cryptographically; `2A` (Sender Keys) reduces group bandwidth from $O(M)$ to $O(1)$ and adds per-message forward secrecy.

### Weakness 3: Self-Signed Relay TLS Certificate on Raw IP (`https://45.77.66.95`)
- **Problem:** Although `v4.6.8+` enforces fail-closed SHA-256 leaf-certificate fingerprint pinning (`PinnedRelayHTTPAdapter`), `VERIFY_TLS = False` is still passed to `requests` to suppress hostname mismatch on the raw IP `45.77.66.95`. If the leaf cert rotates on the server without a simultaneous client pin update, clients fail closed (`ConnectionError`).
- **Candidate Solutions:**
  1. *Option 3A:* Put the relay behind a proper DNS hostname (`relay.fractalops.ai`) with Automated ACME (Let's Encrypt) TLS + SPKI backup pin (`PINNED_SPKI_SHA256_PRIMARY` + `PINNED_SPKI_SHA256_BACKUP`).
  2. *Option 3B:* Bundle a private Root CA (`fractalops-relay-root-ca.pem`) so `VERIFY_TLS` is always `True` AND leaf/SPKI pinning remains active.
- **Selected Optimal Solution (`3A`):** Serve the relay over `https://relay.fractalops.ai` with standard WebPKI (`verify=True`) + dual SPKI pinning (active key + offline backup key) so certificate renewal never bricks deployed clients.

### Weakness 4: Flat Local Capability Token (`bridge.token` Shared Between Desktop UI and Local CLI Scripts)
- **Problem:** While `v4.6.9+` introduced an ephemeral in-memory `_UI_SESSION_SECRET` cookie (`fops_ui_session`) to separate the human browser UI (`local_ui_human`) from `bridge.token` callers (`local_api_agent`), any local script that can read `~/Library/Application Support/FractalOps/bridge.token` (`0600`) gets full `local_api_agent` send capability across all rooms.
- **Candidate Solutions:**
  1. *Option 4A:* Split local tokens into 3 scoped files: `ui_session` (in-memory only), `agent_ro.token` (`GET /api/rooms`, `/api/status`, `/api/messages` only), and `agent_send.token` (scoped to allowed `room_id`s and gated by `_enforce_outbound_dlp_gate`).
  2. *Option 4B:* macOS Keychain ACL / TouchID prompt per agent send.
- **Selected Optimal Solution (`4A`):** Separate read-only polling (`agent_ro.token`) from message sending (`agent_send.token`) + mandatory `_enforce_outbound_dlp_gate()` (shipped in `v4.7.0`) so background pollers never need send credentials.

### Weakness 5: Outbound Data Loss Prevention (DLP) & Cross-Room Context Isolation
- **Problem:** Automated scripts or agents operating across multiple customer chats can accidentally include tokens, file paths, or JSON blobs from Chat A when posting to Chat B (`rowid=235` leaked `"customer_token": "LA4xLG5xLAS5"` into `grp-b519f703`).
- **Candidate Solutions:**
  1. *Option 5A:* Deterministic `_enforce_outbound_dlp_gate(room_id, text)` inside `send_room_message()` scanning for high-entropy secrets, `bridge.token`, `sk-fops-*`, private keys, and known customer tokens (`LA4xLG5xLAS5`).
  2. *Option 5B:* Per-room outbound allowlist binding (`room_id` $\leftrightarrow$ allowed `ag_chat` UUID in `chat_spaces_registry.json`).
- **Selected Optimal Solution (`5A + 5B`, shipped in `v4.7.0`):** `_enforce_outbound_dlp_gate()` blocks sensitive tokens inside `p2p_crypto_daemon.py` (`Test #30`) before any wire encryption occurs.

### Weakness 6: SQLite Concurrency & Unbounded Local Storage Growth (`fractalops_v4.db`)
- **Problem:** `db_connect()` opens SQLite connections without explicitly enabling `PRAGMA journal_mode=WAL` and `PRAGMA busy_timeout=5000` on every connection, and base64 attachments (`::attach ...::`) are stored inline inside `messages.text`, which can bloat `fractalops_v4.db` over months of usage.
- **Candidate Solutions:**
  1. *Option 6A:* Enable `WAL` mode + `busy_timeout=5000` on connection init and move decoded attachments above 32 KB into content-addressed files (`attachments/<sha256>`) with a compact reference in SQLite.
  2. *Option 6B:* Keep inline text storage and run periodic `VACUUM`.
- **Selected Optimal Solution (`6A`):** Content-addressed attachment store + SQLite WAL mode prevents lock contention between the 5s daemon poller, the HTTP API, and external DB inspectors.

### Weakness 7: Update Manifest Single-Maintainer Key Risk vs. Threshold Signing
- **Problem:** `v4.7.0` separates the dedicated offline macOS release signing key (`949d690b...`) and Windows release signing key (`13e61381...`) from hot chat keys, and enforces 2-origin manifest quorum (`verify_cross_origin_manifest_quorum`). However, compromise of a single offline release key could still sign a bundle if both manifest mirrors are updated.
- **Candidate Solutions:**
  1. *Option 7A:* **2-of-3 Multi-Signature Threshold (`M-of-N` Ed25519)** in `manifest.json` requiring signatures from at least 2 independent maintainers (e.g., Kirill's offline macOS key + Dima's offline Windows key) before `verify_and_apply_bundle_bytes()` accepts any update.
  2. *Option 7B:* Sigstore / GitHub Actions OIDC transparency log attestation.
- **Selected Optimal Solution (`7A`):** 2-of-2 / 2-of-3 Ed25519 co-signing (`signatures: [{signer, sig}, ...]`) in `manifest.json` is zero-dependency (pure PyNaCl) and eliminates single-key compromise risk.

### Weakness 8: Relay Metadata Visibility (Social Graph & Timing Correlation on `45.77.66.95`)
- **Problem:** Even though message bodies are End-to-End Encrypted (`Curve25519 + XSalsa20-Poly1305`), the relay sees `from_pubkey`, `to_pubkey`, `timestamp`, and payload byte length.
- **Candidate Solutions:**
  1. *Option 8A:* Sealed Sender (`crypto_box_seal` anonymous outer envelope so `from_pubkey` is encrypted inside the ciphertext and verified by the recipient after decryption, combined with Blinded Sub-Box capability tokens for relay rate-limiting) + PKCS#7 / 4 KB block padding.
  2. *Option 8B:* Mixnet / Tor onion routing.
- **Selected Optimal Solution (`8A` for `v4.8.0`):** `crypto_box_seal` + 4 KB bucket padding hides `from_pubkey` and exact message length from the relay with zero latency penalty.
