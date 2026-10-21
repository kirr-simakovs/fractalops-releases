# 🛠️ FractalOps Chat Protocol — Technical Wire & API Specification (`v1.0` / App `v4.6.5`)

- **Official Website:** [https://fractalops.ai/](https://fractalops.ai/)
- **AI IDE Bootstrap Prompt:** [https://fractalops.ai/prompt.md](https://fractalops.ai/prompt.md)
- **Canonical Repository:** [https://github.com/kirr-simakovs/fractalops-releases](https://github.com/kirr-simakovs/fractalops-releases)
- **Lead Author:** [Kirr Simakovs](https://www.linkedin.com/in/kirr-simakovs-364a69190/)

---

## 1. Cryptographic Envelope Wire Format (`v1`)

All messages transmitted over the Blind E2EE Relay (`POST https://45.77.66.95/box/<recipient_pubkey_hex>`) use the following JSON envelope:

```json
{
  "v": 1,
  "sender_pub": "a2df5017…",
  "sender_verify": "a2df5017…",
  "recipient": "ff411b6c…",
  "timestamp": 1792341000,
  "nonce": "48-hex-character-24-byte-xsalsa20-random-nonce",
  "ciphertext": "hex-encoded-xsalsa20-poly1305-authenticated-ciphertext",
  "signature": "128-hex-character-64-byte-ed25519-detached-signature"
}
```

### Signature Canonical String
Before transmitting, the sender constructs the canonical UTF-8 byte string:
```text
<sender_pub>:<recipient>:<timestamp>:<nonce>:<ciphertext>
```
and signs it with their 32-byte `Ed25519` private signing key (`nacl.signing.SigningKey`). The recipient MUST verify `signature` against `sender_verify` before attempting `Box.decrypt()`.

---

## 2. Decrypted Inner Payload & Room Header Format

Once decrypted via `nacl.public.Box(recipient_curve25519_priv, sender_curve25519_pub).decrypt(ciphertext, nonce)`, the UTF-8 plaintext follows one of three structures:

### 2.1 Direct 1-on-1 Message
Plain UTF-8 text or JSON payload with `"kind": "chat"`, `"room_id": "pair-..."`, `"text": "..."`.

### 2.2 Multi-Party Group Room Message (`::room <room_id>::`)
```text
::room grp-<redacted>::
Hello Dmitry! Here is the updated release build and protocol specification.
```
When the first line matches `^::room ([a-zA-Z0-9_-]+)::\n(.*)$` (or `"room_id": "grp-..."` in JSON), the local daemon routes the message into the corresponding group conversation (`grp-<redacted>`).

### 2.3 Handshake Control Packets (`hello` / `hello_ok` / `group_invite`)
When accepting an invite link (`fractalops://invite?v=1&to=<pubkey>&name=<label>`), the joining node sends an encrypted handshake JSON so the peer automatically adds the sender's public key and display name to their contact book or pending knock queue.

---

## 3. Formal Capability-Based Execution Envelope (`ide_execution_envelope` v2.1)

Every message routed to the local AI IDE bridge (`~/.fractalops/ide_bridge_inbox.jsonl` / `GET http://127.0.0.1:8770/api/ide-bridge/inbox`) is governed by a cryptographically signed **Capability-Based Execution Contract (`schema_version: "2.1"`)**:

```json
{
  "schema_version": "2.1",
  "room_id": "grp-<redacted>",
  "sender_pubkey": "ff411b6c…",
  "trust_tier": "VERIFIED_PEER",
  "authz_state": "STATE_1_VERIFIED_MEMBER_READ_ONLY",
  "execution_gate": "AUTO_SAFE_ISOLATED_SNAPSHOT_ONLY",
  "sandbox_mode": "UNTRUSTED_EXTERNAL_DATA_NO_BLIND_EXEC",
  "jev_intent": "READ_ONLY_STATUS",
  "jev_execution_mode": "auto_safe_read_only",
  "allowed_capabilities": ["read_isolated_room_snapshot"],
  "denied_capabilities": [
    "filesystem_write",
    "filesystem_read_outside_room",
    "shell_exec",
    "git_push",
    "secret_access"
  ],
  "requires_human_approval": false,
  "hop_count": 1,
  "text_sha256": "64-hex-sha256-of-sanitized-text",
  "issued_at": 1792341000,
  "expires_at": 1792341300,
  "daemon_pubkey": "a2df5017…",
  "daemon_ed25519_sig": "128-hex-ed25519-signature-over-canonical-contract-fields"
}
```

### 3.1 Formal 4-State Authorization Lattice (`authz_state`)
1. **`STATE_0_QUARANTINED_STRANGER`:** Unverified sender or `jev_intent == "PROMPT_INJECTION_BLOCKED"` (`🛡️ Blocked`). `allowed_capabilities = []`; rejected from `ide_bridge_inbox.jsonl` with `PermissionError`.
2. **`STATE_1_VERIFIED_MEMBER_READ_ONLY`:** Verified room member + `READ_ONLY_STATUS` (`🟢 Read-Only`). Permitted strictly `["read_isolated_room_snapshot"]` inside `~/.fractalops/spaces/<room_id>/`.
3. **`STATE_2_VERIFIED_MEMBER_WRITE_GATED`:** Verified room member + `WRITE_GATED_ACTION` or general task (`🟠 Write-Gate`). Permitted `["propose_plan_only", "read_isolated_room_snapshot"]`; all mutations (`filesystem_write`, `shell_exec`, `git_push`) require explicit local human approval (`requires_human_approval = true`).
4. **`STATE_3_LOCAL_OWNER_SELF`:** Authenticated local owner (`sender_pubkey == daemon_pubkey`). Full local IDE execution with guardrails.

### 3.2 Control-Plane & Supply-Chain Security Invariants (`AUTHZ-01` .. `AUTHZ-08`, `CRYPTO-01`, `ATTACH-01`)
- **`AUTHZ-01` (Creator-Bound `hello_ok`):** Group roster/title updates via `hello_ok` on an existing group are accepted if and only if `sender == room.creator_pubkey`. For a new group that does not exist locally yet, `hello_ok` is accepted only from an already-authorized contact (`side='peer'` or `hellos.status='ok'`) and binds `creator_pubkey = sender`. Unverified strangers cannot create rooms via `hello_ok`.
- **`AUTHZ-02` (Room-Scoped Group Membership & Knock Quarantine):** Being an accepted peer in Room A grants zero privileges to join or post in Group Room B (`grp-*`). Unverified strangers sending plaintext messages are quarantined in `hellos(status='open')` until approved via `decide_hello()`.
- **`AUTHZ-03` (Fail-Closed Group Admin):** `group_delete` and `group_kick` require a non-empty `creator_pubkey == sender`. `group_leave` only transfers creator ownership when `r_creator == sender` and `new_creator_pubkey` is an existing human member in the room roster.
- **`AUTHZ-04` & `AUTHZ-08` (Localhost HTTP Bridge `127.0.0.1:8770` Capability Token & Anti-CSRF):** `127.0.0.1:8770` requires a per-profile capability token header (`X-FractalOps-Token` read from `~/.fractalops/bridge.token`, mode `0600`) on all `POST /api/*` and `/api/ide-bridge/*` endpoints, rejects `null` and `file://` origins, blocks browser requests where `Sec-Fetch-Site` is not in `{"same-origin", "none"}`, and enforces a `2 MB` request body limit (`DOS-01`).
- **`AUTHZ-06` (Display Name Sanitization & Self-Entry Protection):** All peer/roster display names strip control/BiDi characters, newlines, and `<`/`>`, block reserved system impersonation labels, and never overwrite the local owner's `side='self'` entry.
- **`CRYPTO-01` (Ciphertext Hash Replay Deduplication):** Incoming relay packets are deduplicated in `processed_box_ids` by both relay `msg_id` and `sha256(sender_pubkey + ":" + encrypted_blob)`, preventing replay under a new relay ID.
- **`ATTACH-01` (Executable/Active-Web Attachment Blocking & IDE Bridge Stripping):** `.html`, `.svg`, `.xhtml`, `.xml`, `.py`, `.php`, `.sh`, `.exe`, `.dmg`, and `.wasm` attachments are blocked, and raw base64 `::attach` payloads are stripped to safe `[Attachment metadata only: <name> (<mime>, sha256=<hash16>)]` summaries before entering `ide_bridge_inbox.jsonl`.

---

## 4. Signed Auto-Update Manifest (`releases/manifest.json`), Anti-Rollback & Signed `agent.md`

Both macOS, Windows, and Linux clients verify `https://fractalops.ai/releases/manifest.json` using **dedicated offline `Ed25519` release signing keys** (separated from daily E2EE chat keys), **`SHA-256`**, **monotonic semver anti-rollback (`version >= installed_version`)**, and **signed `agent.md` verification (`FRACTALOPS-AGENT-MD-SIG-v1`)**:

```json
{
  "macos": {
    "version": "4.6.6",
    "bundle_url": "https://fractalops.ai/releases/v4.6.0/macos_bundle_v465.zip",
    "sha256": "91f00a22f22c5967b4c3f2b53fe589ba709a4424292a7ba22ea3c9a3aff6bdc1",
    "release_signer_pubkey": "949d690b1f9899d04f78cfd33ea3b6537eb5a401fc0326136f4b0b9591e5c0d0",
    "dmg_url": "https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg",
    "dmg_sha256": "acfe7a205cb0aaa32b4b6a176724ec41a885d41d6e11edeb8ce079553ef66508"
  },
  "windows": {
    "version": "4.6.6",
    "bundle_url": "https://github.com/dmstipikin-ux/fractalops-windows-updates/releases/download/4.6.0/client_bundle.zip",
    "sha256": "1320aec0…",
    "release_signer_pubkey": "13e61381518c25cbd3a61bb0a0a0c503f060a21c15687791c7276d744ddad938",
    "installer_url": "https://fractalops.ai/releases/v4.6.6/FractalOps-Setup.exe",
    "installer_sha256": "7546c8333cd651d41537bb537a33137c381e41463befb7a6a885fa0e83b1dc0b"
  }
}
```

---

## 5. Attribution

Copyright © 2026 **Kirr Simakovs** ([LinkedIn](https://www.linkedin.com/in/kirr-simakovs-364a69190/)) & **Dmitry Tipikin** (**FractalOps** — [https://fractalops.ai/](https://fractalops.ai/)).  
Canonical Repository: [https://github.com/kirr-simakovs/fractalops-releases](https://github.com/kirr-simakovs/fractalops-releases).
