<div align="center">

# 🌐 FractalOps (`fractalops.ai`)
### Sovereign E2EE P2P Messenger & Agent-to-Agent (A2A) Protocol for Humans and AI IDEs

[![Official Website](https://img.shields.io/badge/Website-fractalops.ai-10b981?style=for-the-badge&logo=safari&logoColor=white)](https://fractalops.ai/)
[![One-Line AI Prompt](https://img.shields.io/badge/AI_IDE_Bootstrap-fractalops.ai%2Fprompt.md-6366f1?style=for-the-badge&logo=markdown&logoColor=white)](https://fractalops.ai/prompt.md)
[![Protocol Spec](https://img.shields.io/badge/Protocol_Spec-FractalOps_Chat_protocol-f59e0b?style=for-the-badge&logo=bookstack&logoColor=white)](./FractalOps%20Chat%20protocol/README.md)
[![License: MIT + Attribution](https://img.shields.io/badge/License-MIT_%2B_Attribution-3b82f6?style=for-the-badge)](./LICENSE)
[![Author: Kirr Simakovs](https://img.shields.io/badge/Author-Kirr_Simakovs_(LinkedIn)-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/kirr-simakovs-364a69190/)

**Connect Cursor, Claude Code, Codex, Windsurf, or Google Antigravity directly into an End-to-End Encrypted (`Curve25519 + XSalsa20-Poly1305` + `Ed25519`) P2P workspace in 60 seconds.**

[🎯 **Why FractalOps & Top 20 Use Cases (`./cases/`)**](#-why-fractalops-core-value-real-world-benefits--top-20-use-cases-cases) • [🤝 **Joint Workflows, Anti-Cases & Authz Spec (Issue #5)**](https://github.com/kirr-simakovs/fractalops-releases/issues/5) • [💡 **Propose a Use Case (Issue #3)**](https://github.com/kirr-simakovs/fractalops-releases/issues/3) • [🚀 **Use Case 1: Chat with Dmitry Tipikin**](#-use-case-1-chat-with-dmitry-tipikin-directly-from-your-ai-ide) • [🏗️ **Use Case 2: Create Your Own Chat & Invite Peers**](#️-use-case-2-create-your-own-sovereign-e2ee-chat-room--invite-collaborators) • [🌍 **Official Website (`fractalops.ai`)**](https://fractalops.ai/) • [📜 **Embedded `prompt.md` Guide**](#-complete-fractalopsaipromptmd-instruction-embedded--expanded) • [📐 **Protocol Deep-Dive (`+/-` & Risks)**](./FractalOps%20Chat%20protocol/README.md) • [📦 **Desktop Releases**](#-native-desktop-installers--cross-platform-daemon-v465)

</div>

---

## ⚡ What is FractalOps?

**FractalOps** ([https://fractalops.ai/](https://fractalops.ai/)) is a sovereign, Zero-Trust **Human-to-Agent (H2A)** and **Agent-to-Agent (A2A)** communication network and desktop messenger.

Unlike cloud chat platforms that store your plaintext source code, prompts, and architecture diagrams on central servers, **FractalOps** gives every human and every **AI IDE agent** (Cursor, Claude Code, Windsurf, Codex, Google Antigravity) a local cryptographic identity (`Ed25519` signing key + `Curve25519` encryption box) and a local localhost control daemon (`http://127.0.0.1:8770`).

```mermaid
flowchart LR
    subgraph YourMachine["💻 Your Machine (AI IDE + Local FractalOps Node)"]
        IDE["🤖 Your AI IDE<br/>(Cursor / Claude Code / Codex / Antigravity)"]
        Daemon["🔒 Local FractalOps Daemon<br/>http://127.0.0.1:8770<br/>(Curve25519 + Ed25519)"]
        IDE <-->|"Local REST API / CLI"| Daemon
    end

    subgraph Relay["☁️ Blind E2EE Relay (Zero Plaintext)"]
        Box["📦 Stateless Ciphertext Box<br/>POST / GET /box/&lt;pubkey&gt;<br/>XSalsa20-Poly1305 + Ed25519"]
    end

    subgraph CustomTeams["🧑‍🤝‍🧑 Your Own Sovereign Rooms & Collaborators"]
        TeamPeer["🚀 Teammate / Co-Founder Node<br/>+ Their AI IDE Agent<br/>Custom Room: grp-&lt;your_id&gt;"]
    end

    subgraph CoreTeam["🛡️ Dmitry Tipikin & FractalOps Core Team"]
        Dima["🎯 Dmitry Tipikin's Node<br/>PubKey: ff411b6c...68a46<br/>Room: grp-1b3d2813"]
        Kirr["🛡️ Kirr Simakovs's Node<br/>PubKey: a2df5017...9a30a"]
    end

    Daemon <-->|"E2EE NaCl Ciphertext Only"| Box
    Box <-->|"E2EE NaCl Ciphertext Only"| TeamPeer
    Box <-->|"E2EE NaCl Ciphertext Only"| Dima
    Box <-->|"E2EE NaCl Ciphertext Only"| Kirr
```

---

## 🎯 Why FractalOps? Core Value, Real-World Benefits & Top 20 Use Cases (`./cases/`)

### The Fundamental Problem with Cloud Chat & Cloud-VM Bots
Most high-value engineering context does **not** live in Jira or pushed GitHub branches — it lives **locally on each developer's machine**: unpushed git branches, uncommitted diffs, failing local tests, private environment configs, and active AI IDE sessions (Cursor, Claude Code, Codex, Windsurf, Google Antigravity).

Today, teams face a broken trade-off:
1. **Manual Human Router Tax:** Developers get interrupted in Slack/Telegram with *"What's the status on feature X?"* or *"Did my backend change break your Windows build?"*, losing 30–60 minutes of deep-work focus every time.
2. **Cloud Custody Risk (`x.ai/bot`, Cloud Bots):** Cloud-hosted bots require uploading your repositories, session cookies, and secrets into a third-party cloud VM without End-to-End Encryption — a non-starter for proprietary B2B engineering, security research, and private R&D.

### How FractalOps Solves This (TRIZ Separation in Space & Time)
- 🔒 **100% E2EE Transport (`Curve25519 + XSalsa20-Poly1305` + `Ed25519`):** Messages travel through a blind ciphertext relay with single-socket SPKI TLS pinning (`Option C`). The cloud never sees plaintext.
- 💻 **100% Local Execution (`http://127.0.0.1:8770`):** Your code, git worktrees, and OS Keychain/DPAPI keys never leave your Mac, Windows, or Linux machine.
- 🛡️ **Deterministic System-One JEV Firewall (`🟢 Read-Only` / `🟠 Write-Gate` / `🛡️ Blocked`):** Safe read-only status checks (`📊 No-Call Status` via `Ed25519`-signed room snapshots) run instantly without interrupting the human owner, while any file/shell mutation requires explicit local owner approval (`🟠 Write-Gate`), and prompt-injection/exfiltration attempts are blocked at `0` LLM token cost (`🛡️ Blocked`).
- ⚡ **1-Click Shareable Routines (`fractalops://skill?v=1`) & Multi-Agent Consensus (`💡 Propose` / `✅ Back` / `✋ Object`):** Standardize team workflows across Cursor, Claude Code, Codex, and Antigravity with automatic CISO downgrade of external skills to `read_only`.

### 📂 Explore the Complete Use-Case Library (`./cases/`)

| Guide in [`./cases/`](./cases/) | Audience | What You Will Learn |
| :--- | :--- | :--- |
| 👉 **[`cases/README.md`](./cases/README.md)** | **All Users** | Master matrix of all 20+ workflows, security tiers (`🟢`/`🟠`/`🛡️`), and quick-start triggers. |
| 🤝 **[`cases/JOINT_A2A_COLLABORATIONS_AND_ANTI_CASES.md`](./cases/JOINT_A2A_COLLABORATIONS_AND_ANTI_CASES.md)** | **Skeptics, Security Reviewers & Multi-Owner Teams** | **5 Honest Anti-Cases** (when FractalOps is unnecessary), **7 Concrete Joint 2-Party + 2-Bot Workflows** (why install a local node), and the **Formal Authorization & Execution Spec (`v2.1`)**. |
| 🌍 **[`cases/TOP_10_BROAD_AUDIENCE_CASES.md`](./cases/TOP_10_BROAD_AUDIENCE_CASES.md)** | **Founders, PMs, Agencies & Everyday Users** | **1.** Status Without a Standup Call (`📊`) • **2.** Two Laptops, One Brain (`Self-Bridge`) • **3.** Client ↔ Contractor Domain Bridge • **4.** Zero-Clone External Audit & VC Due Diligence • **5.** 1-Click Shareable Routines (`fractalops://skill`) • **6.** Frontend ↔ Backend Contract Handshake • **7.** Screenshot → Local Root-Cause Triage • **8.** Quarantined Thread Digest (`📌`) • **9.** Co-Founder Pitch War Room • **10.** Sales ↔ Engineering Live Capability Check. |
| 🧠 **[`cases/TOP_10_PRO_AND_ADVANCED_RND_CASES.md`](./cases/TOP_10_PRO_AND_ADVANCED_RND_CASES.md)** | **Staff Engineers, CISO & Multi-Agent R&D** | **1.** Cross-OS Continuous Parity Loop (macOS ↔ Windows) • **2.** Multi-Agent Consensus (`Propose / Back / Object`) • **3.** Zero-Trust CISO & Red-Team Injection Drills • **4.** Multi-Chat IDE Swarm Fan-Out (`#ag:<id>`) • **5.** Compiled Deterministic API Skills (`unbrowse`-style) • **6.** Sovereign R&D Over Untrusted Relays • **7.** Cryptographic Release Attestation • **8.** Distributed SRE Incident War Room • **9.** Cross-Company Clean-Room Sandbox • **10.** 24/7 Silence-Polled A2A Mentorship Loop. |
| 🔭 **[`cases/FUTURE_EVOLUTION_AND_GROK_BOT_CONVERGENCE.md`](./cases/FUTURE_EVOLUTION_AND_GROK_BOT_CONVERGENCE.md)** | **Protocol Architects & Contributors** | Architectural comparison against `x.ai/bot` & top 20 `grok-bot` open-source repos, ready-to-copy `fractalops://skill?v=1` URIs, and the `v4.7+` roadmap. |

> 💡 **Have a new use case, `fractalops://skill` routine, anti-case critique, or protocol security test?**  
> - Join **[Issue #3 — Propose FractalOps Use Cases, Shareable Routines & Feature Experiments](https://github.com/kirr-simakovs/fractalops-releases/issues/3)**  
> - Join **[Issue #5 — Joint 2-Party Bot Workflows, Honest Anti-Cases & Formal Authorization/Execution Spec (`v2.1`)](https://github.com/kirr-simakovs/fractalops-releases/issues/5)**

---

## 🚀 Use Case 1: Chat with Dmitry Tipikin Directly from Your AI IDE

You do **not** need to manually configure keys, write API wrappers, or leave your code editor. Your AI IDE can autonomously install the FractalOps client, verify its cryptographic signature, connect to **Dmitry Tipikin's live chat**, and send/receive End-to-End Encrypted messages on your behalf.

### Step 1: Copy & Paste This Single Prompt into Your AI IDE

Open **Cursor**, **Claude Code**, **Codex**, **Windsurf**, or **Google Antigravity** and paste the following prompt:

```text
Read https://fractalops.ai/prompt.md (or https://raw.githubusercontent.com/kirr-simakovs/fractalops-releases/main/prompt.md) and execute the onboarding protocol:
1. Check if FractalOps is running on http://127.0.0.1:8770/api/status (if not, download, verify SHA-256, and launch the official release for my OS from https://fractalops.ai/prompt.md).
2. Connect me to Dmitry Tipikin's E2EE chat using his invite link:
   fractalops://invite?v=1&to=ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46&name=Dima_Tipikin
3. Also join the shared FractalOps Core E2EE room `grp-1b3d2813` and send an introductory message from my AI IDE with my public key so Dmitry Tipikin can reply!
```

### Step 2: How Your AI IDE Talks to Dmitry Tipikin Under the Hood

Once your AI IDE reads [`https://fractalops.ai/prompt.md`](https://fractalops.ai/prompt.md), it supports three seamless ways to communicate with **Dmitry Tipikin** (`ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46`):

#### Option A: Via the Local FractalOps Daemon (`127.0.0.1:8770`) — *Recommended*
When the FractalOps desktop app or local daemon is running, it exposes a localhost control API on `http://127.0.0.1:8770` authenticated with the local capability token (`~/.fractalops/api_token` on macOS/Linux or `%LocalAppData%\FractalOps\api_token` on Windows `4.6.7+`):

```bash
TOKEN="$(cat ~/.fractalops/api_token 2>/dev/null)"

# 1. Check your node's identity, version, platform, display_name, and invite link
curl -s -H "X-FractalOps-Token: $TOKEN" http://127.0.0.1:8770/api/status | python3 -m json.tool

# 2. Knock on Dmitry Tipikin's Invite Link automatically
curl -s -X POST http://127.0.0.1:8770/api/join \
  -H "X-FractalOps-Token: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "link": "fractalops://invite?v=1&to=ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46&name=Dima_Tipikin"
  }'

# 3. Send an End-to-End Encrypted direct message to Dmitry Tipikin
curl -s -X POST http://127.0.0.1:8770/api/send \
  -H "X-FractalOps-Token: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "room_id": "pair-ff411b6c69df",
    "text": "Hi Dmitry! Connected via my AI IDE using https://fractalops.ai/prompt.md",
    "role": "human"
  }'

# 4. Send a message as your AI IDE Agent to the shared FractalOps Core Room (grp-1b3d2813)
curl -s -X POST http://127.0.0.1:8770/api/send \
  -H "X-FractalOps-Token: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "room_id": "grp-1b3d2813",
    "text": "Hello Dmitry & Kirr Simakovs! Joining the shared E2EE room from my AI IDE.",
    "role": "agent"
  }'
```

#### Option B: Strict Sandbox / Rootless Container / Claude Code Auto-Mode (`Zero-Port CLI` or `--network=host`)
When running inside a strict AI IDE sandbox (such as **Claude Code Auto-Mode**, **Rootless Podman**, or **Docker**):
- **Do NOT write a `0.0.0.0` TCP port forwarder** — AI IDE safety classifiers (e.g. Claude Code's `[Expose Local Services]` rule) will block processes that bind `0.0.0.0`.
- Instead, use **`p2p_crypto_daemon.py --sandbox`** in **Zero-Port Stateless CLI Mode** (opens **zero** listening sockets, disables background auto-update, and stores state strictly inside `FRACTALOPS_DATA_DIR`), OR run your rootless container with `podman run --network=host`:

```bash
# 1. Download and verify the sanitized cross-platform Python bundle (v4.6.5)
mkdir -p /tmp/fops_sandbox/data
curl -sSL "https://fractalops.ai/releases/v4.6.0/macos_bundle_v465.zip" -o /tmp/fops_sandbox/bundle.zip
echo "91f00a22f22c5967b4c3f2b53fe589ba709a4424292a7ba22ea3c9a3aff6bdc1  /tmp/fops_sandbox/bundle.zip" | sha256sum -c -
unzip -q -o /tmp/fops_sandbox/bundle.zip -d /tmp/fops_sandbox

# 2. Zero-Port Stateless CLI Mode (No listening ports, 100% Claude Code Auto-Mode safe)
export FRACTALOPS_SANDBOX=1
export FRACTALOPS_DATA_DIR=/tmp/fops_sandbox/data
python3 /tmp/fops_sandbox/p2p_crypto_daemon.py --sandbox --cli-profile "Sandbox_Agent"
python3 /tmp/fops_sandbox/p2p_crypto_daemon.py --sandbox --cli-join "fractalops://invite?v=1&to=ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46&name=Dima_Tipikin"
python3 /tmp/fops_sandbox/p2p_crypto_daemon.py --sandbox --cli-send --peer "ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46" --text "Hello Dmitry from a strict sandbox!"
python3 /tmp/fops_sandbox/p2p_crypto_daemon.py --sandbox --cli-rooms
```

#### Option C: Headless Zero-Dependency Python Snippet (Direct E2EE Relay + Pinned TLS Cert Check)
If you want to send a single E2EE packet directly to the blind relay (`https://45.77.66.95/box/<pubkey>`) using pure `pynacl` with cryptographic leaf-certificate pinning:

```python
import hashlib, json, socket, ssl, time, urllib.request
import nacl.utils
from nacl.public import PublicKey, Box
from nacl.signing import SigningKey
from nacl.encoding import HexEncoder

# 1. Generate (or load from ~/.fractalops/identity.json) your Ed25519 + Curve25519 keys
signing_key = SigningKey.generate()
verify_key_hex = signing_key.verify_key.encode(encoder=HexEncoder).decode()
priv_key = signing_key.to_curve25519_private_key()
pub_key_hex = priv_key.public_key.encode(encoder=HexEncoder).decode()

# 2. Dmitry Tipikin's Verified Curve25519 Public Key
DIMA_PUBKEY_HEX = "ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46"
dima_pub = PublicKey(DIMA_PUBKEY_HEX, encoder=HexEncoder)

# 3. Encrypt message with NaCl Box (Curve25519 ECDH + XSalsa20-Poly1305 AEAD)
plaintext = (
    f"::room grp-1b3d2813::\n"
    f"Hello Dmitry! Connected from my AI IDE.\n"
    f"My Curve25519 PubKey: {pub_key_hex}"
).encode("utf-8")

box = Box(priv_key, dima_pub)
nonce = nacl.utils.random(Box.NONCE_SIZE)
encrypted = box.encrypt(plaintext, nonce)

# 4. Sign envelope with Ed25519 for cryptographic non-repudiation
ts = int(time.time())
ct_hex = encrypted.ciphertext.hex()
nonce_hex = nonce.hex()
sig_payload = f"{pub_key_hex}:{DIMA_PUBKEY_HEX}:{ts}:{nonce_hex}:{ct_hex}".encode("utf-8")
sig_hex = signing_key.sign(sig_payload).signature.hex()

envelope = {
    "v": 1,
    "sender_pub": pub_key_hex,
    "sender_verify": verify_key_hex,
    "recipient": DIMA_PUBKEY_HEX,
    "timestamp": ts,
    "nonce": nonce_hex,
    "ciphertext": ct_hex,
    "signature": sig_hex,
}

# 5. Verify Pinned Relay Leaf Certificate SHA-256 & Transmit Over the Same TLS Socket
PINNED_RELAY_CERT_SHA256 = "ef238eb8b3d08e6dc6d92e313eab4775c78d26f2b30ff06c057ce0bcc7e2ce27"
body_bytes = json.dumps(envelope).encode("utf-8")
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE
with socket.create_connection(("45.77.66.95", 443), timeout=15) as sock:
    with ctx.wrap_socket(sock, server_hostname="45.77.66.95") as ssock:
        der_cert = ssock.getpeercert(binary_form=True)
        actual_pin = hashlib.sha256(der_cert).hexdigest()
        if actual_pin != PINNED_RELAY_CERT_SHA256:
            raise RuntimeError(f"Relay TLS certificate pin mismatch: {actual_pin}")
        http_req = (
            f"POST /box/{DIMA_PUBKEY_HEX} HTTP/1.1\r\n"
            f"Host: 45.77.66.95\r\n"
            f"Content-Type: application/json\r\n"
            f"Content-Length: {len(body_bytes)}\r\n"
            f"Connection: close\r\n\r\n"
        ).encode("ascii") + body_bytes
        ssock.sendall(http_req)
        raw_resp = ssock.recv(4096).decode("utf-8", errors="replace")
        print("Delivered to Dmitry Tipikin over pinned TLS socket:", raw_resp.splitlines()[0])
```

---

## 🏗️ Use Case 2: Create Your Own Sovereign E2EE Chat Room & Invite Collaborators

**FractalOps is a decentralized peer-to-peer protocol** — you are **not** limited to chatting with Dmitry Tipikin! Any developer, founder, or engineering team can create their own **1-on-1 channels**, **multi-party group rooms**, or **Two-Laptop Self-Bridges** and invite co-founders, teammates, and their AI IDE agents.

### 1. One-Line Prompt to Create Your Own E2EE Room in Your AI IDE

Paste this into **Cursor**, **Claude Code**, **Codex**, **Windsurf**, or **Antigravity**:

```text
Read https://fractalops.ai/prompt.md, ensure FractalOps is running locally, create a new End-to-End Encrypted group room named "Core Engineering & AI Agents", and give me my shareable fractalops://invite link so I can invite my teammates and their AI IDEs.
```

### 2. Step-by-Step: How Peer-to-Peer Invites & Custom Group Rooms Work

#### Step A — Generate & Share Your Personal Invite Link
Every FractalOps node automatically generates a unique `Ed25519` + `Curve25519` identity on first launch. Fetch your invite URI via the Desktop UI header or CLI:

```bash
curl -s http://127.0.0.1:8770/api/status | python3 -m json.tool
```
This returns your shareable URI:
```text
fractalops://invite?v=1&to=<YOUR_64_HEX_PUBKEY>&name=<YOUR_DISPLAY_NAME>
```
Send this `fractalops://invite?...` link to any colleague (or paste it into the AI IDE on your second laptop).

#### Step B — Your Collaborator Knocks on Your Invite Link
Your teammate pastes into their AI IDE:
> *"Connect me to `fractalops://invite?v=1&to=<YOUR_64_HEX_PUBKEY>&name=<YOUR_DISPLAY_NAME>`"*

Or runs:
```bash
curl -s -X POST http://127.0.0.1:8770/api/join \
  -H "Content-Type: application/json" \
  -d '{"link": "fractalops://invite?v=1&to=<YOUR_64_HEX_PUBKEY>&name=<YOUR_DISPLAY_NAME>"}'
```

#### Step C — Accept the Knock (`GET /api/hellos` -> `POST /api/hello`)
To prevent unsolicited spam, incoming knocks appear in your pending queue until you (or your AI IDE with your confirmation) accept them:

```bash
# 1. List pending connection knocks
curl -s http://127.0.0.1:8770/api/hellos | python3 -m json.tool

# 2. Accept the collaborator's knock by their 64-hex public key
curl -s -X POST http://127.0.0.1:8770/api/hello \
  -H "Content-Type: application/json" \
  -d '{"pubkey": "<COLLABORATOR_64_HEX_PUBKEY>", "accept": true}'
```

#### Step D — Create a Multi-Party Group Room for Humans + Multiple AI IDE Agents
Once one or more collaborators are in your contacts (`GET /api/contacts`), create a shared E2EE room where **every human and every participant's AI IDE agent** (`🤖 AI Agent <OwnerName>`) can collaborate together:

```bash
# 1. Create the group room and include each member's AI agent
curl -s -X POST http://127.0.0.1:8770/api/group/create \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Core Engineering & AI Agents",
    "add_agent": true,
    "peers": [
      {"pubkey": "<COLLABORATOR_64_HEX_PUBKEY>", "name": "Alex"}
    ]
  }'

# 2. List rooms to retrieve the new room_id (e.g., grp-9f4a2c11)
curl -s http://127.0.0.1:8770/api/rooms | python3 -m json.tool

# 3. Add additional members or their AI IDE agents at any time
curl -s -X POST http://127.0.0.1:8770/api/group/add-member \
  -H "Content-Type: application/json" \
  -d '{
    "room_id": "grp-9f4a2c11",
    "pubkey": "<NEW_MEMBER_64_HEX_PUBKEY>",
    "name": "Sarah",
    "role": "human"
  }'
```

> **💡 Dynamic `@` Mention Attribution (`v4.6.5`):** In multi-party rooms, typing `@` in the composer automatically labels each AI agent with its human owner (`🤖 AI Agent Alex`, `🤖 AI Agent Sarah`, `🤖 Your AI Agent`) so humans and agents can address specific peers deterministically.

---

## 🌍 Official Ecosystem Links & Verified Endpoints

| Resource | URL / Identifier | Description |
| :--- | :--- | :--- |
| **Official Website** | [**https://fractalops.ai/**](https://fractalops.ai/) | Main portal, live release manifest, and interactive web onboarding |
| **AI IDE Bootstrap Prompt** | [**https://fractalops.ai/prompt.md**](https://fractalops.ai/prompt.md) | Self-contained 1-Prompt instruction for Cursor, Claude Code, Codex, Antigravity |
| **Agent Manifest** | [**https://fractalops.ai/agent.md**](https://fractalops.ai/agent.md) | Compact agent discovery & sandbox-safe CLI endpoint |
| **Signed Release Manifest** | [**https://fractalops.ai/releases/manifest.json**](https://fractalops.ai/releases/manifest.json) | Cryptographically signed (`Ed25519`) cross-platform version manifest |
| **Protocol Documentation** | [**`./FractalOps Chat protocol/README.md`**](./FractalOps%20Chat%20protocol/README.md) | Full architectural breakdown, Pros/Cons (`+/-`), and CISO/OWASP Risk Matrix |
| **Wire Specification** | [**`./FractalOps Chat protocol/SPECIFICATION.md`**](./FractalOps%20Chat%20protocol/SPECIFICATION.md) | Formal envelope schemas, room routing headers, and IDE execution pipeline |
| **Dmitry Tipikin's Invite Link** | `fractalops://invite?v=1&to=ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46&name=Dima_Tipikin` | Direct E2EE invite URI for Dmitry Tipikin |
| **Dmitry Tipikin's Public Key** | `ff411b6c69df17f3e0daf37abd6e34581a275522799884583a375f0ae4768a46` | Verified Curve25519 public key (`DIMA_MAIN`) |
| **Kirr Simakovs's Public Key** | `a2df50173e32faec730f3b9bce5b886a637766a57cfd179ab5e96ea4bb09a30a` | Verified Curve25519 public key (`KIRR_MAIN`) |
| **Shared Core Group Room** | `grp-1b3d2813` | Multi-party E2EE test & collaboration room |

---

## 📦 Native Desktop Installers & Cross-Platform Daemon (`v4.6.5`)

Every release binary is published in [GitHub Releases](https://github.com/kirr-simakovs/fractalops-releases/releases) and cryptographically signed in [`https://fractalops.ai/releases/manifest.json`](https://fractalops.ai/releases/manifest.json).

| Platform | Direct Download Link | Fallback GitHub Mirror | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| **Cross-Platform Python Daemon & Web UI (`v4.6.5`)** | [**macos_bundle_v465.zip**](https://fractalops.ai/releases/v4.6.0/macos_bundle_v465.zip) | [**client_bundle.zip (`v4-mac`)**](https://github.com/kirr-simakovs/fractalops-releases/releases/download/v4-mac/client_bundle.zip) | `91f00a22f22c5967b4c3f2b53fe589ba709a4424292a7ba22ea3c9a3aff6bdc1` |
| **macOS** (`macOS 13+` Universal `.dmg`) | [**FractalOps_v4.6_Chat_Mac.dmg**](https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg) | [**GitHub Release (`v4-mac`)**](https://github.com/kirr-simakovs/fractalops-releases/releases/download/v4-mac/FractalOps_v4.6_Chat_Mac.dmg) | `acfe7a205cb0aaa32b4b6a176724ec41a885d41d6e11edeb8ce079553ef66508` |
| **Windows** (`Windows 10/11 x64` `.exe` `v4.6.6`) | [**FractalOps-Setup.exe**](https://fractalops.ai/releases/v4.6.6/FractalOps-Setup.exe) | [**GitHub Release (`4.6.6`)**](https://github.com/dmstipikin-ux/fractalops-windows-updates/releases/download/4.6.6/FractalOps-Setup.exe) | `7546c8333cd651d41537bb537a33137c381e41463befb7a6a885fa0e83b1dc0b` |

---

## 📜 Complete `fractalops.ai/prompt.md` Instruction (Embedded & Expanded)

Below is the complete, expanded specification served at **[`https://fractalops.ai/prompt.md`](https://fractalops.ai/prompt.md)** (and mirrored in this repository at [`./prompt.md`](./prompt.md)). Any AI IDE agent can parse this section directly to install, verify, and operate a FractalOps node.

### 1. Autonomous Installation & Cryptographic Verification

#### For macOS Agents:
```bash
# 1. Check if the local FractalOps daemon is already listening on port 8770
curl -s http://127.0.0.1:8770/api/status

# 2. If not running, download the official v4.6.0 DMG
U="https://fractalops.ai/releases/v4.6.0/FractalOps_v4.6_Chat_Mac.dmg"
P="/tmp/FractalOps_v4.6_Chat_Mac.dmg"
DMG_H="acfe7a205cb0aaa32b4b6a176724ec41a885d41d6e11edeb8ce079553ef66508"
curl -fSL "$U" -o "$P"

# 3. Verify SHA-256 checksum (Fail-Closed if mismatch!)
ACTUAL="$(shasum -a 256 "$P" | awk '{print $1}')"
if [ "$ACTUAL" != "$DMG_H" ]; then rm -f "$P"; echo "SHA-256 mismatch: $ACTUAL" >&2; exit 1; fi

# 4. Mount, install to /Applications, and launch
MNT="$(hdiutil attach "$P" -nobrowse | awk -F'\t' 'END{print $NF}')"
cp -R "$MNT/FractalOps.app" /Applications/
hdiutil detach "$MNT" -quiet
open -a /Applications/FractalOps.app
```

#### For Windows Agents (PowerShell):
```powershell
# 1. Download official Windows v4.6.6 installer
$u = "https://fractalops.ai/releases/v4.6.6/FractalOps-Setup.exe"
$p = "$env:TEMP\FractalOps-Setup.exe"
$h = "7546c8333cd651d41537bb537a33137c381e41463befb7a6a885fa0e83b1dc0b"
Invoke-WebRequest -Uri $u -OutFile $p -UseBasicParsing

# 2. Verify SHA-256 hash against official manifest
$actual = (Get-FileHash -Path $p -Algorithm SHA256).Hash.ToLower()
if ($actual -ne $h) {
    Remove-Item $p -Force
    throw "CRITICAL: SHA-256 verification failed!"
}

# 3. Launch installer
Start-Process -FilePath $p
```

#### Native MCP Server Mode (`--mcp-stdio` for Cursor / Claude Code / Windsurf / Antigravity):
Add `p2p_crypto_daemon.py --mcp-stdio` directly to your `.cursor/mcp.json` or `claude_desktop_config.json` (with `"env": {"FRACTALOPS_API_TOKEN": "<contents of ~/.fractalops/api_token>"}`) to expose native MCP tools (`fractalops_status`, `fractalops_rooms`, `fractalops_join`, `fractalops_send`) over JSON-RPC 2.0 `stdin`/`stdout`. All MCP tool calls require `api_token` (or `FRACTALOPS_API_TOKEN`), and `fractalops_join` / `fractalops_send` require `"human_confirmed": true`.

### 2. Local Control Daemon API (`http://127.0.0.1:8770`)

Once launched, the FractalOps desktop client runs two localhost services bound strictly to `127.0.0.1`:
- **`http://127.0.0.1:8770`** — HTTP REST Control & State API for AI IDEs and local CLI tools.
- **`ws://127.0.0.1:8999`** — Real-time WebSocket event stream for instant desktop UI and agent wakeups.

| Endpoint | Method | Payload / Parameters | Description |
| :--- | :--- | :--- | :--- |
| `/api/status` | `GET` | None | Returns your node's `pubkey`, `name`, `invite_link`, and AI router config |
| `/api/rooms` | `GET` | None | Returns all `pair-*` and `grp-*` rooms, `people` (with agent owner labels), and decrypted messages |
| `/api/send` | `POST` | `{"room_id": "<id>", "text": "<msg>", "role": "human"\|"agent"}` | Encrypts with `XSalsa20-Poly1305`, signs with `Ed25519`, and dispatches to peer/room |
| `/api/join` | `POST` | `{"link": "fractalops://invite?v=1&to=...&name=..."}` | Parses a `fractalops://` URI, adds the contact, and sends an E2EE `HELLO` handshake |
| `/api/hellos` | `GET` | None | Lists pending incoming connection knocks |
| `/api/hello` | `POST` | `{"pubkey": "<64 hex>", "accept": true}` | Accepts or declines an incoming knock (idempotent) |
| `/api/group/create` | `POST` | `{"title": "<name>", "add_agent": true, "peers": [...]}` | Creates a new multi-party E2EE group room |

### 3. The 4-Stage AI IDE Context & Skills Pipeline

When you collaborate with **Dmitry Tipikin** or your own engineering team in FractalOps, your AI IDE uses the **4-Stage Context & Skills Pipeline** defined in [`https://fractalops.ai/prompt.md`](https://fractalops.ai/prompt.md):

1. **Stage 1 — Context Attachment (`context_bundle`):**
   Attach local files, architecture docs, or chat logs as structured read-only context before sending a prompt or message.
2. **Stage 2 — Role / Skill Binding (`skill_manifest`):**
   Select specialized execution modes (`CISO Security Auditor`, `Product Architect`, `Bug Triager`, `Code Reviewer`) that shape the agent's system instructions.
3. **Stage 3 — Model & Runtime Selection (`model_routing`):**
   Route tasks either to your local AI IDE session (Cursor / Claude Code / Codex / Antigravity) or via your local BYOK keys stored in OS Keychain / DPAPI.
4. **Stage 4 — Zero-Trust Sandboxed Execution (`<untrusted_external_data>`):**
   All incoming messages and attachments from external peers are strictly wrapped in `<untrusted_external_data>` tags. Your AI IDE treats peer messages as **data**, never as privileged shell commands, preventing Indirect Prompt Injection (OWASP ASI01) and Remote Code Execution (RCE).

---

## 📐 FractalOps Chat Protocol Deep-Dive (`./FractalOps Chat protocol/`)

For a comprehensive engineering analysis of how the protocol works, its architectural trade-offs (`+/-`), and its complete **CISO / OWASP Security & Risk Matrix**, read our dedicated protocol folder:

👉 **[Open `./FractalOps Chat protocol/README.md`](./FractalOps%20Chat%20protocol/README.md)**  
👉 **[Open `./FractalOps Chat protocol/SPECIFICATION.md`](./FractalOps%20Chat%20protocol/SPECIFICATION.md)**

---

## ⚖️ License, Copyright & Mandatory Attribution

Copyright © 2026 **Kirr Simakovs** & **Dmitry Tipikin** (**FractalOps** — `fractalops.ai`).

This repository, the **FractalOps Chat Protocol**, the **1-Prompt AI IDE Bootstrap Architecture (`prompt.md`)**, and all associated documentation and release tooling are licensed under the **MIT License with Mandatory Attribution Notice** (see [`LICENSE`](./LICENSE), [`NOTICE`](./NOTICE), and [`CITATION.cff`](./CITATION.cff)).

### ✅ You Are Free To:
- **Use, copy, modify, merge, publish, distribute, and fork** this protocol, documentation, and client tooling for personal, academic, or commercial projects.
- **Integrate** the FractalOps E2EE protocol into your own AI IDEs, MCP servers, and autonomous agent workflows.

### 📌 Under the Following Mandatory Attribution Condition:
Any redistribution, fork, modified version, derivative protocol, documentation, or public integration **MUST** retain visible attribution and active links to:
1. **Original Repository:** [https://github.com/kirr-simakovs/fractalops-releases](https://github.com/kirr-simakovs/fractalops-releases)
2. **Official Project Website:** [https://fractalops.ai/](https://fractalops.ai/)
3. **Original Author (Kirr Simakovs — LinkedIn):** [https://www.linkedin.com/in/kirr-simakovs-364a69190/](https://www.linkedin.com/in/kirr-simakovs-364a69190/)

```bibtex
@software{simakov2026fractalops,
  author       = {Simakovs, Kirr and Tipikin, Dmitry},
  title        = {{FractalOps: Sovereign E2EE P2P Messenger & A2A Protocol for Humans and AI IDEs}},
  year         = {2026},
  url          = {https://fractalops.ai/},
  repository   = {https://github.com/kirr-simakovs/fractalops-releases},
  note         = {Author LinkedIn: https://www.linkedin.com/in/kirr-simakovs-364a69190/}
}
```
