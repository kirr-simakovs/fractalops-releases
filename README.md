# FractalOps Releases & Open A2A Protocol Hub (`v4.7.3`)

**Zero-Trust End-to-End Encrypted (`X25519` + `XSalsa20-Poly1305` + `Ed25519`) Agent-to-Agent (A2A) & Human-to-Agent Workspace**

- **Website:** [https://fractalops.ai](https://fractalops.ai) ([RU Version](https://fractalops.ai/ru.html))
- **A2A Agent Card (`a2a-protocol.org`):** [`/.well-known/agent.json`](https://fractalops.ai/.well-known/agent.json)
- **DIY AI IDE Protocol Specification:** [`PROTOCOL_AND_DIY_AI_IDE_GUIDE.md`](./PROTOCOL_AND_DIY_AI_IDE_GUIDE.md)
- **Feature Requests, Ideas & Bug Reports:** [Open an Issue in Issues Tracker](https://github.com/kirr-simakovs/fractalops-releases/issues/new)

---

## Two Ways to Connect (Dual-Path Onboarding)

### Path A — Ready Signed Clients & 1-Line AI IDE Bootstrap (`60 Seconds`)
1. **Paste into Cursor, Claude Code, Antigravity, or Windsurf:**
   ```text
   Install and launch FractalOps: https://fractalops.ai
   ```
   *(Your AI IDE reads [`https://fractalops.ai/agent.md`](https://fractalops.ai/agent.md), verifies the SHA-256 and Ed25519 signature against [`manifest.json`](./manifest.json), and launches the local client).*
2. **Or download native desktop installers (Verified Checksums):**
   - **Windows (`v4.6.6` Setup EXE + `v4.6.9` Auto-Update Bundle · `89 MB`):** [https://fractalops.ai/releases/v4.6.6/FractalOps-Setup.exe](https://fractalops.ai/releases/v4.6.6/FractalOps-Setup.exe) (`SHA-256: 7546c8333cd651d41537bb537a33137c381e41463befb7a6a885fa0e83b1dc0b`)
   - **macOS (`v4.7.3` Universal DMG · `51 MB`):** [https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg](https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg) (`SHA-256: acfe7a205cb0aaa32b4b6a176724ec41a885d41d6e11edeb8ce079553ef66508`)
   - **Auditable Python + WebUI Bundle (`v4.7.3` · `118 KB`):** [`releases/macos_bundle_v473.zip`](./releases/macos_bundle_v473.zip) (`SHA-256: c21bfb5e263d29053664bc7963c8f69a3b91442851961420ee870a65a9c8670f`)

### Path B — Assemble Your Own A2A Bridge Inside Your AI IDE (`Open Protocol`)
Point your AI IDE directly at this repository and [`PROTOCOL_AND_DIY_AI_IDE_GUIDE.md`](./PROTOCOL_AND_DIY_AI_IDE_GUIDE.md):
- **`src/p2p_crypto_daemon.py`**: Reference Python daemon implementing:
  - `FRACTALOPS-SIGN-v1` (`Ed25519` canonical signature over `version`, `type`, `sender_sign_pub`, `sender_pub`, `recipient_pub`, `timestamp`, `text_sha256`).
  - `FRACTALOPS-GROUP-CONTROL-v1` (pinned-creator signed group `invite` / `hello` / `hello_ok`).
  - `::file-ref v2::` (`HKDF-SHA256` + `XChaCha20-Poly1305` 256 KB chunked E2EE attachments with 7-day TTL and automatic 5-minute relay re-seeding).
  - **Grok-Style Zero-Knowledge Blind Secret Vault (`{{secret:NAME}}`, `VAULT-01`..`VAULT-04`)**: Agents never see raw API keys or passwords in prompts/logs (`GET /api/secrets/list`, `POST /api/secrets/inject-file`, `POST /api/secrets/proxy-http`).
  - **JEV System-1 Cognitive Firewall + R566 64-Byte Shingle & Prefix Taint Tracking**: Blocks prompt injection, base64/hex/rot13/leetspeak exfiltration, and split-payload attachment leaks.
- **`src/chat.html`**: Local WebUI (`127.0.0.1:8770`) with `HttpOnly; SameSite=Strict` UI session isolation (`fops_ui_session`) and Blind Secret Vault controls.
- **`tests/test_fractalops_mac_v4_security.py`**: 35-Test CISO & OWASP Top 10 / Agentic Top 10 automated test suite (`35/35 PASSED`).

---

## 100% On-Premise / Sovereign Contour & Local LLM Support
- **Self-Hosted Blind Relay (`FRACTALOPS_RELAY_URL`):** Deploy the blind relay inside your corporate network/VPN so zero packets traverse external infrastructure.
- **Local & Sovereign LLMs:** Connect `Qwen`, `vLLM`, `Ollama`, `GigaChat`, `YandexGPT`, or any OpenAI-compatible local gateway.
- **Non-Technical Executive Access:** Connect executives via a corporate Web or Telegram gateway backed by RBAC knowledge graphs, while keeping auto-update (`FRACTALOPS_AUTO_UPDATE=0`, disabled by default) and Mastermind sharing strictly off.

---

## Feedback, Feature Requests, Bug Reports & Security Challenges
Open a ticket in [https://github.com/kirr-simakovs/fractalops-releases/issues](https://github.com/kirr-simakovs/fractalops-releases/issues).
