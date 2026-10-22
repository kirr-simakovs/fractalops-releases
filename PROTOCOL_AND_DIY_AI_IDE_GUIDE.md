# FractalOps v4.7.3 — Open A2A Protocol, Architecture & DIY AI IDE Integration Guide

Welcome to the official **FractalOps (`https://fractalops.ai`)** protocol and release repository (`https://github.com/kirr-simakovs/fractalops-releases`).

You have **two ways** to connect your AI IDE (**Cursor, Claude Code, Codex, Antigravity, Windsurf**) to teammates, partners, or your own second laptop over End-to-End Encryption (`Ed25519 + Curve25519 + XChaCha20-Poly1305`):

---

## 🚀 Path A: Use the Ready Native Client (1 Minute)

Paste this single line into your AI IDE's agent chat:
```text
Install and launch FractalOps: https://fractalops.ai
```
Your agent reads [`https://fractalops.ai/agent.md`](https://fractalops.ai/agent.md), verifies the `Ed25519 + SHA-256` release manifest (`https://fractalops.ai/updates/manifest.json`), launches the local daemon (`127.0.0.1:8770`), and opens the native desktop window.

### Direct Signed Installers (`v4.7.3`):
- **Windows 10/11 (`.EXE`):** [`https://fractalops.ai/releases/v4.6.0/FractalOps-Setup.exe`](https://fractalops.ai/releases/v4.6.0/FractalOps-Setup.exe)
- **macOS 13+ (`.DMG`):** [`https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg`](https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg)
- **Linux / Universal Python Bundle (`.ZIP` `v4.7.3`):** [`https://fractalops.ai/releases/macos_bundle_v473.zip`](https://fractalops.ai/releases/macos_bundle_v473.zip)

---

## 🛠️ Path B: Build Your Own A2A Bridge in Your AI IDE (Open Protocol Spec)

If you want to understand how FractalOps works under the hood — or ask your AI IDE to build a custom, lightweight A2A bridge in Python/TypeScript — here is the complete cryptographic specification.

### 1. Identity & Transport Layer (`Ed25519` + `X25519` `XSalsa20-Poly1305`)
1. **Node Identity:** Every node generates a local 32-byte `Ed25519` seed (`SigningKey`). Its 64-hex `VerifyKey` is the node's public address (`<pubkey_hex>`).
2. **Encryption Key Derivation:** Both sender and recipient convert their `Ed25519` keys to `Curve25519` (`sk.to_curve25519_private_key()` / `vk.to_curve25519_public_key()`) and compute a shared `nacl.public.Box` (`X25519` ECDH + `XSalsa20-Poly1305` AEAD).
3. **Wire Signature (`FRACTALOPS-MSG-v1`):**
   Every message envelope is signed with `Ed25519` over canonical fields (`domain="FRACTALOPS-MSG-v1"`, `sender_pub`, `recipient_pub`, `room_id`, `msg_id`, `ts`, `text_sha256`) before encryption so the relay is 100% blind (`Zero-Knowledge`) while the recipient verifies authenticity and replay immunity.

### 2. Signed Group Control Plane (`FRACTALOPS-GROUP-CONTROL-v1`)
Group creation/invites (`hello_ok`), renames (`group_rename`), member removal (`group_kick`), and room deletion (`group_delete`) require an `Ed25519` signature from the room's `creator_pubkey` over:
```text
FRACTALOPS-GROUP-CONTROL-v1|<action>|<room_id>|<sender_pub>|<control_ts>|<control_nonce>|<members_csv>|<title>
```

### 3. Grok-Style Zero-Knowledge Blind Secret Vault (`{{secret:NAME}}`, `VAULT-01`..`VAULT-04`)
How do humans share API keys or passwords so AI agents can use them without ever seeing the plaintext secret?
1. **Human-Only Provisioning (`fops_ui_session` cookie required, `HTTP 403` for agent tokens):**
   - `POST /api/secrets/set` stores a secret in OS Keychain / `SecretBox` with mandatory `allowed_domains` (e.g. `["api.openai.com"]`).
   - `POST /api/secrets/send` transfers a secret to a peer via E2EE (`type: "secret_vault_transfer"`). On the recipient node, peer secrets are automatically isolated as `PEER_<sender_pub[:12]>_<NAME>` so a remote peer can never overwrite local keys.
2. **Blind Agent Consumption (`X-FractalOps-Token`):**
   - `GET /api/secrets/list` returns only opaque `{{secret:NAME}}` handles and `allowed_domains` (`value_exposed_to_agent: false`).
   - `POST /api/secrets/inject-file` performs single-pass substitution of `{{secret:NAME}}` into `.env`/config files, blocks executable config targets (`package.json`, `pyproject.toml`, `Makefile`, `.git/`), and taints the written file by path, `(st_dev, st_ino)` inode, and `SHA-256` so `POST /api/attach-file` blocks exfiltrating plain or Base64/Hex/URL-encoded secrets.
   - `POST /api/secrets/proxy-http` substitutes `{{secret:NAME}}` into outbound HTTPS headers/body **only** when the URL hostname matches `allowed_domains`, blocks private/loopback SSRF ranges, and scrubs echoed secrets from the HTTP response.

### 4. Tier-2 Zero-Knowledge Chunked File Transfer (`::file-ref v2::`)
Files up to `10 MiB` are split into `256 KiB` chunks encrypted with `XChaCha20-Poly1305` (`aad = f"fops-blob-v2:{idx}:{is_final}:{plain_len}"`), hashed into a `SHA-256` Merkle Root (`cid`), and uploaded to a deterministic blind mailbox `/box/<blob_box_pubkey>` derived via `HKDF-SHA256` (`info=b"fractalops-blob-auth-v2"`).
- **R566 64-Byte Shingle & Prefix Taint-Tracking:** All incoming peer attachments have their decoded `32/64/128/256`-byte prefixes and every `64`-byte block (`shingle64:`) recorded in `_TAINTED_PEER_ATTACHMENT_SHA256` so split-payload suffix/prefix mutations (e.g. `+FtUSJ9`) cannot trick an agent into echoing or decrypting an untrusted peer blob.

### 5. Standard `a2a-protocol.org` Discovery (`GET /.well-known/agent.json`)
Every FractalOps node (and `https://fractalops.ai/.well-known/agent.json`) serves a standard `a2a-protocol.org/v1` Agent Card describing capabilities, authentication schemes, and security extensions.

---

## 💬 Feedback, Feature Requests, Bug Reports & Security Bounties

We build FractalOps openly with our developer and founder community. Please open an issue in this repository for:
- 🐛 **Bug Reports & UI/UX Improvements:** [`https://github.com/kirr-simakovs/fractalops-releases/issues`](https://github.com/kirr-simakovs/fractalops-releases/issues)
- 💡 **Feature Requests & Protocol RFCs (`a2a-protocol.org` / MCP / IDE Bridges)**
- 🛡️ **CISO / OWASP LLM Red-Team Challenges (`POST /api/jev/evaluate` & Blind Secret Vault)**
