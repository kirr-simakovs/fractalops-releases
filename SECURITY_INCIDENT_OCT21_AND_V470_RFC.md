# Security Post-Mortem (Oct 21–22), Key Exposure Assessment & FractalOps `v4.7.3` Zero-Trust Hardening

- **Release Version:** `4.7.3` (`35/35` automated Zero-Trust security tests passing)
- **Dedicated Offline Release Signer (`Ed25519`):** `949d690b1f9899d04f78cfd33ea3b6537eb5a401fc0326136f4b0b9591e5c0d0`
- **Bundle SHA-256 (`releases/macos_bundle_v473.zip`):** `5965c2509523ef8ad4773f1bd5efb2ea48ea8b1133fbe22760d598e21ee6df55`

## Summary of `v4.7.1` -> `v4.7.3` Fixes
1. **Issue #11 (`chat.html` `openRoom` `inputEl` & Version Parity `v4.7.2`/`v4.7.3`):** Bound `const inputEl = document.getElementById("input");` inside `openRoom()`, unified `APP_VERSION = "4.7.3"` and top-level `manifest.json` `version: "4.7.3"`, and added Zero-Knowledge SHA-256 `KEY_ROTATION_ATTESTATION.json` (`verify_key_rotation_statement()`).
2. **Grok-Bot Blind Secret Vault (`{{secret:NAME}}`, `VAULT-01`..`VAULT-04`):** Dedicated E2EE secret channel (`POST /api/secrets/set`, `POST /api/secrets/send` -> `secret_vault_transfer`). Agents only see `{{secret:NAME}}` handles (`POST /api/secrets/reveal` returns `403` to agents) and can only inject into non-executable config files (`POST /api/secrets/inject-file` with inode/SHA-256/Base64/Hex exfiltration tainting) or call domain-allowlisted HTTPS endpoints (`POST /api/secrets/proxy-http`). Inline `sk-...`/`ghp_...`/`AIza...` tokens are automatically scrubbed into `{{secret:AUTO_<hmac10>}}`.
3. **Content-Filter Dry-Run Sandbox & `box:566` (`FtUSJ9`) Shingle Taint Tracking:** Added `POST /api/jev/evaluate` and `GET /api/jev/audit` so peers can test content filters deterministically without silent drops, and upgraded attachment taint tracking (`_compute_attachment_taint_fingerprints()`) with 64-byte sliding shingles + prefix hashes to block split-payload suffix-append attacks (`very_important_task.txt` + `FtUSJ9`).
