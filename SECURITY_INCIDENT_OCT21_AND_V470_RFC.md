# Security Post-Mortem (Oct 21 Incident), Key Exposure Assessment & FractalOps `v4.7.1` Zero-Trust Hardening

**Date:** October 21, 2026  
**Authors:** Kirill & FractalOps Security Engineering  
**Special Thanks / Red-Team Credit:** `@bisqet` and `@Agent bisqet` (`Issue #6`, `R173`, `R175`, `R177`, `rowid=215/216`, `rowid=235`)  
**Release Version:** `v4.7.1` (`32/32` automated Zero-Trust security tests passing)

---

## 1. Executive Summary & Root Cause of the October 21 Incident (`rowid=235`)

On October 21, 2026 at 18:04 (+04), a cross-workspace synchronization script (`publish_issue13_mastermind_and_e2ee_sync.py`) triggered two simultaneous regressions:
1. **Cross-Workspace Identity Overwrite (`5aec3c82…` $\to$ `a2df5017…`):** The script overwrote `~/Library/Application Support/FractalOps/ed25519.priv` with the primary workspace identity `a2df5017…` without sending a membership migration packet to the security review group room (`grp-<redacted>`), where our node had originally joined under `5aec3c82…`.
   - On peers still running `v4.6.5` (which lacked the `AUTHZ-02` `members` check added in `v4.6.6+`), messages from `a2df5017…` rendered in the group timeline even though `a2df5017…` was not in that room's `members` table.
   - Replies encrypted by group peers were addressed to `5aec3c82…`, which our local daemon had stopped polling after `ed25519.priv` was overwritten.
2. **Unscoped Room Broadcast & Internal Token Exposure:** The sync script iterated across all rooms in `fractalops_v4.db` and sent a JSON status payload containing an internal workspace blueprint marker (`<redacted-token>`) into the external security group room.
3. **Public GitHub Issue #7 Redaction Violation:** When our automated reporting script published the initial incident summary to `Issue #7`, it copied raw 64-hex chat public keys (`161c92ad…`, `a2df5017…`, `5aec3c82…`), a researcher's surname, and internal room IDs into a public GitHub issue — violating our own M14 sterilization policy and `@bisqet`'s disclosure rule (*"Pubkeys above are abbreviated to 8 hex chars + … only"*).

---

## 2. Cryptographic & Operational Risk Assessment of the Public Key Exposure

### 2.1 Did Any Private Signing Key (`ed25519.priv`) Leak?
- **No (`0%` private key compromise).** Every 64-hex string that appeared in `Issue #7` (`a2df5017…`, `5aec3c82…`, `161c92ad…`, `ff411b6c…`, `9867f245…`) is a **32-byte Ed25519 public verification key** (which also maps via `VerifyKey.to_curve25519_public_key()` to the peer's X25519 encryption public key).
- The corresponding 32-byte private signing seeds in `~/Library/Application Support/FractalOps/ed25519.priv` and macOS Keychain (`ai.fractalops.ed25519`) **never left the local host**.
- **Cryptographic impact:** No attacker can forge `X-Signature` headers, sign release manifests, or decrypt Curve25519 `Box` ciphertexts addressed to those keys.

### 2.2 Why Full 64-Hex Chat Public Keys Must Still Be Treated as Sensitive Routing Capabilities
Even though an Ed25519 public key is mathematically public, in the FractalOps v4 relay architecture a peer's 64-hex chat public key serves as their **direct relay mailbox routing address** (`POST /box/<pubkey_hex>`):
1. **Unauthenticated Relay Mailbox Flooding / Spam (`Issue #6` `F3`):** Any party who scrapes a full 64-hex chat public key from a public GitHub page can `POST` arbitrary `hello` knocks or junk ciphertexts to `/box/<pubkey_hex>` (the relay verifies that the *sender* signed the request with *some* key, not that the sender is authorized to message the recipient).
2. **Relay Traffic Metadata Correlation (`Issue #6` `F2`):** Knowing a user's full 64-hex chat public key allows cross-correlating public GitHub identities with E2EE chat personas.
3. **Why Agents & Scripts Missed It:**
   - Our `_enforce_outbound_dlp_gate()` in `v4.7.0` only hooked `send_room_message()` inside `p2p_crypto_daemon.py`, NOT outbound GitHub REST API calls in `09_Management/_CLI_AI_APIs/github/gh_utils.py`.
   - Worse, `p2p_crypto_daemon.py` and `test_fractalops_mac_v4_security.py` previously contained the literal forbidden token string inside the DLP check itself, which then got committed to the public repository!

---

## 3. Remediations Shipped in FractalOps `v4.7.1` (`32/32 PASSED`)

### 3.1 Zero-Knowledge SHA-256 Outbound DLP Gate & Mandatory GitHub M14 Sterilization
- **Zero-Knowledge Token Matching (`OUTBOUND_DLP_FORBIDDEN_TOKEN_SHA256`):** `_enforce_outbound_dlp_gate()` no longer stores any literal token string in source code or tests. Instead, candidate tokens are hashed with `SHA-256` and compared against `OUTBOUND_DLP_FORBIDDEN_TOKEN_SHA256` (`738975c3…`).
- **Mandatory Public GitHub Sterilization (`sterilize_public_github_text()` & `gh_utils.py`):** Every public GitHub API `POST`/`PATCH` and repository markdown/code export passes through `sterilize_public_github_text()`, which automatically truncates any non-allowlisted 64-hex string to `8-hex…`, redacts SHA-256-matched tokens to `<redacted-token>`, and scrubs `grp-*` / `pair-*` room IDs (`Test #32`).
- **Complete Repository & Git History Purge:** All historical commits on `kirr-simakovs/fractalops-releases` containing full 64-hex chat keys or tokens have been replaced with a clean root commit on `main`, and all issues/PRs/comments have been scrubbed.

### 3.2 Zero-Downtime Signed Key Rotation (`FRACTALOPS-KEY-ROTATE-v1`), Multi-Key Ring & Blinded Box IDs
- **Multi-Key Ring Polling (`get_all_local_signing_keys()`):** `p2p_crypto_daemon.py` now maintains `~/Library/Application Support/FractalOps/keyring/*.priv` (`0600`) and polls/decrypts relay mailboxes for **both** the current primary identity key and all archived/rotated keys (`Test #31`).
- **Dual-Signed Key Rotation (`rotate_chat_identity_key()` & `verify_and_apply_key_rotation()`):** When a user rotates their chat key, the daemon generates a `{"type": "key_rotation", "domain": "FRACTALOPS-KEY-ROTATE-v1", "old_pubkey": ..., "new_pubkey": ..., "ts": ..., "sig_old": ..., "sig_new": ...}` certificate signed by **both** the old Ed25519 key and the new Ed25519 key. Peers verify both signatures before atomically migrating `members.pubkey` and `rooms.creator_pubkey` (`Test #31`).
- **Blinded Relay Mailbox IDs (`derive_blinded_box_id()`):** Derives a domain-separated `SHA-256("fractalops-blinded-box-v2:" + capability_salt + ":" + pubkey_hex)` routing identifier so relay mailbox addresses are decoupled from raw identity verification keys.

### 3.3 Cryptographic Taint Tracking on Echoed Peer Attachments (`R177` `funny_little_command.md` Defense)
- In `funny little chat`, `@bisqet` tested whether an attacker could bypass the `is_self` trust check by asking our agent to echo back an encrypted peer attachment (`funny_little_command.md`, `EOE1` PBKDF2-HMAC-SHA256 / AES-256-GCM with password `FUNNY`), download it from our own `self` message, and execute the decrypted instruction.
- **Defense-in-Depth (`Test #29` & `Test #31`):**
  1. **`SELF_AGENT` Provenance Separation:** All messages sent by the local agent (`role="agent"` or `origin_channel="local_api_agent"`) receive `trust_tier = "SELF_AGENT"` and are hard-blocked (`PermissionError`) from entering `ide_bridge_inbox.jsonl`.
  2. **Permanent Attachment SHA-256 Tainting (`_TAINTED_PEER_ATTACHMENT_SHA256`):** Every inline attachment (`::attach ...::`) ingested from any remote peer has its base64 payload SHA-256 recorded in `_TAINTED_PEER_ATTACHMENT_SHA256`. If that attachment is ever echoed back in a `self` message (even with `role="human"`), `enqueue_ide_bridge_message()` detects `has_tainted_peer_attachment(text) == True` and raises `PermissionError("Self-echoed message contains tainted peer attachment (R177 defense)")`.

### 3.4 Complete Closure Matrix for `@bisqet` `Issue #6` (`F1`–`F15`) & `rowid=215/216`

| ID | Finding (`Issue #6` / `rowid=215/216`) | Status in `v4.7.1` (`src/p2p_crypto_daemon.py`) |
| :--- | :--- | :--- |
| **F1** | `/api/ide-bridge/ack` unauthenticated | **FIXED:** Guarded by `self._bridge_auth_ok()` (`X-FractalOps-Token` constant-time `hmac.compare_digest`). |
| **F2 / F3** | Relay `GET`/`POST /box/<pub>` auth & spam | **FIXED (Client) + Staged (Relay):** Envelope v2 (`X-Signer-Pubkey`, `X-Timestamp`, `X-Nonce`, `X-Signature`), 7-day freshness window, and `derive_blinded_box_id()`. |
| **F4 / F5** | Petname spoofing & `hello_ok` member injection | **FIXED:** `_get_established_petname()` preserves verified petnames; `hello_ok` only admits `f_pub` (verified outer sender), never unverified third-party entries (`Test #23`). |
| **F6 / F7** | Unauthenticated `group_rename` / `group_delete` / `group_kick` | **FIXED:** Verified against `rooms.creator_pubkey` (only verified `creator_pubkey` can delete, kick, or rename; creator cannot be kicked). |
| **F8** | Non-member message acceptance (`AUTHZ-02`) | **FIXED:** `ingest_inbox_item()` verifies `SELECT 1 FROM members WHERE room_id=? AND lower(pubkey)=?` before accepting any group message (`Test #4`, `Test #23`). |
| **F9** | Keychain `security -w <secret>` in `ps aux` argv | **FIXED:** `_keychain_set()` pipes `add-generic-password ...` via stdin to `security -i`, keeping secrets out of process argv. |
| **F10 / F11** | Replay / stale timestamp / `sqlite3` permissions | **FIXED:** Dual deduplication (`msg_id` + `blob_sha256` in `processed_box_ids`), 7-day timestamp window, `PRAGMA journal_mode=WAL`, and `0600` on `fractalops_v4.db` (`-wal`/`-shm`). |
| **F12 / F13** | `.svg` / `.html` attachment XSS & `/agent.md` exposure | **FIXED:** `.svg`, `.html`, `.js`, `.py`, `.exe` blocked in `BLOCKED_ATTACH_EXTS`; `SAFE_RASTER_IMAGE_MIMES` enforced; `/agent.md` requires `_bridge_auth_ok()`. |
| **F14** | `FALLBACK_MANIFEST_URL` served stale `4.6.7` & quorum not wired | **FIXED:** `verify_cross_origin_manifest_quorum()` wired into `check_and_apply_auto_update()`, and GitHub Release tag `v4-mac` asset `manifest.json` updated to `4.7.1`. |
| **F15** | Hot chat keys in `TRUSTED_RELEASE_SIGNING_PUBKEYS` | **FIXED:** Hot chat keys removed; only offline release keys `949d690b1f9899d04f78cfd33ea3b6537eb5a401fc0326136f4b0b9591e5c0d0` (macOS) and `13e61381518c25cbd3a61bb0a0a0c503f060a21c15687791c7276d744ddad938` (Windows) trusted (`Test #29`). |

<!-- Verified by FractalOps v4.7.1 32/32 Zero-Trust Suite -->
