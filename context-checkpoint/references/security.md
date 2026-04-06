# Encryption, Privacy, and Context Lifespan

## Overview

This reference covers: GPG/password encryption of checkpoint files, TTL-based auto-expiry, and privacy handling for sensitive conversations.

---

## Encrypted Save (`/save encrypt`)

### Workflow

1. User runs `/save encrypt` (or `/save encrypt gpg` for GPG, `/save encrypt password` for symmetric).
2. AI assembles the full checkpoint `.md` file in memory (same as standard save).
3. AI outputs the file for download and instructs the user to encrypt it locally.
4. AI NEVER stores, logs, or echoes the encryption key.

### Symmetric Password Encryption (AES-256)

Instruct the user to run after download:

```bash
# Encrypt
openssl enc -aes-256-cbc -pbkdf2 -in context-checkpoint_v1.md -out context-checkpoint_v1.md.enc

# Decrypt (on resume)
openssl enc -d -aes-256-cbc -pbkdf2 -in context-checkpoint_v1.md.enc -out context-checkpoint_v1.md
```

### GPG Encryption

Instruct the user to run:

```bash
# Encrypt with GPG key
gpg --encrypt --recipient user@example.com context-checkpoint_v1.md
# Output: context-checkpoint_v1.md.gpg

# Decrypt on resume
gpg --decrypt context-checkpoint_v1.md.gpg > context-checkpoint_v1.md
```

### What the AI Does

- Generates the plaintext `.md` file and presents it for download
- Appends encryption instructions at the bottom of the file before download
- Adds a header line to the file: `ENCRYPTION: required — do not share without decrypting`
- Reminds the user: "Store your key separately from the checkpoint file"

### What the AI Never Does

- Store, echo, or log any password or GPG key
- Attempt to perform encryption itself (browser/client environments are untrusted)
- Accept a key pasted by the user in the chat (warn if they do and advise them to revoke it)

---

## TTL-Based Auto-Expiry (`/expiry Nd`)

### Setting TTL

User runs `/expiry 7d` before or alongside `/save`. The AI embeds an expiry timestamp in the checkpoint header:

```
TTL: 2026-04-12T00:00:00Z
TTL-ACTION: warn-and-block
```

### Resume with Expiry Check

On resume, the AI reads the `TTL` field and compares to the current date:

- If within TTL: load normally
- If expired: display warning and refuse to load

```
WARNING: This checkpoint expired on 2026-04-12. Loading is blocked to protect potentially stale or sensitive content.
To override: type /resume force (adds a note in the session that context may be outdated)
To delete: delete the .md file locally
```

### Supported TTL Values

| Input | Meaning |
|-------|---------|
| `1d` | 1 day from save time |
| `7d` | 7 days |
| `30d` | 30 days |
| `never` | No expiry (default) |
| `session` | Expires when current browser session ends (note: AI cannot enforce this — remind user) |

### TTL Tampering Detection

The TTL field is included in the SHA-256 checksum (Section 10). If the TTL is edited manually, the checksum will mismatch on load. The AI warns: "Checksum mismatch detected — TTL or content may have been modified."

---

## Privacy Guidelines for Sensitive Sessions

For medical, legal, financial, or personal conversations:

1. Always recommend `/save encrypt` — state this explicitly if the topic is sensitive
2. Suggest the user use a short TTL (e.g., `7d`) for sensitive checkpoints
3. In the checkpoint file itself, add a privacy header:

```
SENSITIVITY: HIGH
Note: This checkpoint contains potentially sensitive information. Encrypt before sharing. Do not upload to public pastebins or cloud storage without encryption.
```

4. In the Continuation Prompt (Section 6), do NOT reproduce raw sensitive data verbatim. Refer to it abstractly: "Refer to the medical history details in Section 3 of the attached checkpoint."

---

## Pastebin / Temporary URL Resume

For mobile users who want a short resume code instead of a file:

1. AI generates the checkpoint content as usual
2. AI instructs the user to paste the content to a user-controlled encrypted pastebin (e.g., PrivateBin, 0bin, or self-hosted)
3. The pastebin generates a URL and decryption key
4. AI generates a 6-character alphanumeric session code that maps to the URL + key:

```
Resume code: 3F9K2A
Maps to: https://privatebin.example.com/?abc123 (key: xyz789)
Store this code — AI cannot retrieve it for you.
```

5. In the new chat, user types `/resume 3F9K2A`
6. AI asks the user to paste the URL and key (it does not store them) and fetches the content with user permission

Note: The AI itself cannot host URLs. This feature requires the user to have a pastebin service. The AI provides the instructions and formatting only.
