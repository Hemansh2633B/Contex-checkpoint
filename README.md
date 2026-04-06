<div align="center">

<img src="assets/banner.svg" alt="context-checkpoint banner" width="100%"/>

<br/>

![Version](https://img.shields.io/badge/version-2.0-1f6feb?style=flat-square\&logo=github\&logoColor=white)
![Spec](https://img.shields.io/badge/agentskills.io-compatible-238636?style=flat-square)
![Models](https://img.shields.io/badge/Claude%20%7C%20GPT--4%20%7C%20Gemini%20%7C%20Llama-cross--model-a371f7?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-e3b341?style=flat-square)
![Status](https://img.shields.io/badge/status-stable-3fb950?style=flat-square)

**An AI Agent Skill that saves, compresses, versions, encrypts, and resumes your chat sessions — across windows, devices, and AI models.**

[Installation](#-installation) · [Commands](#-commands) · [Features](#-features) · [Architecture](#-architecture-overview) · [File Format](#-checkpoint-file-format)

</div>

---

# 🧠 What Is This?

`context-checkpoint` is an **Agent Skill** for the agentskills.io ecosystem that prevents **context loss in long AI conversations**.

Instead of losing your work when token limits are reached, this skill converts your session into a:

* 📄 Structured `.md` checkpoint
* 🔁 Resumable execution state
* 🔀 Cross-model compatible prompt
* 🔐 Optionally encrypted artifact

---

# ⚡ Why This Exists (Problem → Solution)

## ❌ The Problem

* AI chats **silently die at token limits**
* No native way to **persist structured state**
* Switching models = **context loss**
* Long sessions = **token inefficiency**

## ✅ The Solution

This skill turns conversations into:

* **Portable state files**
* **Executable resumes**
* **Model-agnostic memory**

You don’t just save chats — you **checkpoint workflows**.

---

# 🚀 What Makes This Different

Unlike basic chat export tools:

* 🧠 Semantic compression (not naive summarization)
* 🔁 Actionable resumes (execution-ready)
* 🔐 Encryption with zero key exposure
* 🔀 Cross-model continuation (Claude ↔ GPT ↔ Llama)
* 📊 Token usage intelligence
* 🛡️ Self-healing context with checksum validation

This is effectively **Git for AI conversations**.

---

# 🎬 30-Second Demo

```bash
/health        # Check token usage
/save          # Create checkpoint

# New chat (any model):
# Paste continuation prompt

✅ Same state restored
✅ No repetition
✅ Work continues instantly
```

Optional:

```bash
/resume light   # ultra-fast resume
```

---

# 📦 Installation

## Recommended

1. Download `context-checkpoint.zip`
2. Upload to your AI skill manager
3. Done — auto-activates near token limit

## Manual

```bash
git clone https://github.com/your-username/context-checkpoint.git
mv context-checkpoint ~/.skills/
```

---

# ⌨️ Commands (Execution-Oriented)

| Command            | Mode     | Description                     |
| ------------------ | -------- | ------------------------------- |
| `/save`            | Standard | Balanced compression checkpoint |
| `/save light`      | Fast     | Minimal size, fastest resume    |
| `/save encrypt`    | Secure   | Encrypted output                |
| `/resume`          | Full     | Full context reconstruction     |
| `/resume light`    | Fast     | Critical-state only             |
| `/resume [code]`   | Remote   | Resume via short code           |
| `/bookmark "name"` | Snapshot | Named checkpoint                |
| `/list bookmarks`  | Index    | Show saved points               |
| `/diff v2 v3`      | Compare  | Version differences             |
| `/merge`           | Combine  | Merge multiple chats            |
| `/recover_last`    | Recovery | Emergency restore               |
| `/export pdf`      | Export   | Report generation               |
| `/health`          | Monitor  | Token dashboard                 |
| `/expiry 7d`       | Security | Auto-expiry                     |

---

# 🧩 All 20 Advanced Capabilities (v2.0)

✔ Intelligent compression
✔ Selective forgetting
✔ Code hashing (SHA-256)
✔ Cross-model adapters
✔ Encryption (AES / GPG)
✔ TTL auto-expiry
✔ Auto-save + versioning
✔ Diff system
✔ Bookmark system
✔ Token health dashboard
✔ Resume modes (full/light/code)
✔ Collaborative merge
✔ Pending task execution
✔ Voice/multimodal anchoring
✔ Self-healing context
✔ Emergency recovery
✔ Resume via short code
✔ Context analytics export
✔ Adaptive learning thresholds
✔ Master index across sessions

---

# 🏗️ Architecture Overview

```
[Conversation]
     ↓
[Token Analyzer] → Dashboard
     ↓
[Compression Engine]
     ↓
[Checkpoint Builder]
     ↓
[.md File + Checksum]
     ↓
[Resume Engine]
     ↓
[Same / New Model]
```

### Core Systems

* **Compression Engine** → removes noise, hashes large code
* **Checkpoint Builder** → structured 10-section export
* **Resume Engine** → multi-mode reconstruction
* **Integrity Layer** → checksum + redundancy repair

---

# 🔁 Resume Experience

When resuming:

1. AI reads continuation prompt
2. Injects context (full or light)
3. Rebuilds:

   * objectives
   * decisions
   * artifacts
   * pending tasks
4. Continues execution immediately

No repetition. No drift.

---

# ✨ Features

## 💾 Intelligent Compression

* Drops filler messages
* Summarizes low-value threads
* Hashes large code blocks

## 🔀 Cross-Model Compatibility

Works across:

* GPT
* Claude
* Gemini
* Llama

Includes adapter prompts per model.

## 🔐 Encrypted Context Vault

* AES-256 / GPG support
* No key exposure
* TTL expiry protection

## 🔖 Versioning & Bookmarks

* Auto versioning (v1, v2, v3…)
* Named checkpoints
* Diff tracking

## 📊 Token Dashboard

* Live usage tracking
* Breakdown by type
* Predicts limit exhaustion

## 🛡️ Self-Healing

* SHA-256 verification
* Redundancy block repair
* Corruption detection

## 🔗 Collaborative Merge

* Combine multiple chats
* Resolve conflicts interactively

## 📤 Export System

* PDF reports
* Timelines
* Quizzes
* Mermaid diagrams

## 🤖 Adaptive Learning

* Learns user behavior
* Adjusts auto-save threshold
* Optimizes compression

---

# 📄 Checkpoint File Format

```
# Context Checkpoint

## 1. Objectives
## 2. Key Decisions
## 3. Artifacts
## 4. Current State
## 5. Open Issues
## 6. Continuation Prompt
## 7. Pending Actions
## 8. Bookmarks
## 9. Redundancy Block
## 10. Checksum
```

---

# 🔐 Security & Privacy

* No key storage by AI
* Encryption is local-only
* TTL enforced via checksum
* Sensitive mode tagging

---

# 📁 Project Structure

```
context-checkpoint/
├── SKILL.md
└── references/
    ├── compression.md
    ├── cross-model.md
    ├── security.md
    ├── versioning.md
    ├── resume-modes.md
    ├── dashboard.md
    ├── analytics.md
    └── recovery.md
```

---

# 🧪 Power User Tips

* Use `/save light` for rapid iteration
* Use `/bookmark` before major changes
* Use `/diff` to track decisions
* Use `/merge` for parallel workflows
* Use `/export timeline` for documentation

---

# 📋 Requirements

* agentskills-compatible AI platform
* Optional:

  * `openssl` or `gpg`
  * `pandoc` for exports

---

# 🤝 Contributing

1. Update relevant reference file
2. Add command in `SKILL.md`
3. Update README
4. Validate skill

---

# 📄 License

MIT © 2026

---

<div align="center">

⭐ If this saved your session, star the repo.

Built for people who refuse to lose context.

</div>
