# Context Health Dashboard

## When to Show

- Automatically when token usage is estimated at >= 80%
- When user types `/health`
- Always before any `/save` operation

---

## Dashboard Format

Render the dashboard as an ASCII block. Always include:

```
╔══════════════════════════════════════════════════════╗
║           CONTEXT HEALTH DASHBOARD                   ║
╠══════════════════════════════════════════════════════╣
║ Tokens used:    7,200 / 8,192    (88%)               ║
║ [████████████████████░░░░░░]  88%                    ║
║                                                      ║
║ Messages:       42 total                             ║
║ Session start:  2026-04-05 10:00                     ║
║ Elapsed:        ~4h 45m                              ║
╠══════════════════════════════════════════════════════╣
║ TOKEN BREAKDOWN (top consumers)                      ║
║  Code blocks:     3,100  (43%)  [████████░░░░░░░░░░] ║
║  AI responses:    2,200  (31%)  [██████░░░░░░░░░░░░] ║
║  User messages:   1,100  (15%)  [███░░░░░░░░░░░░░░░] ║
║  Tool outputs:      800  (11%)  [██░░░░░░░░░░░░░░░░] ║
╠══════════════════════════════════════════════════════╣
║ PACE ANALYSIS                                        ║
║  Last 10 messages added:  1,200 tokens (16%)         ║
║  At current pace, limit reached in: ~1-2 messages    ║
╠══════════════════════════════════════════════════════╣
║ RECOMMENDATION                                       ║
║  [!] Save now — limit imminent                       ║
║  Options: /save  /save light  /save encrypt          ║
╚══════════════════════════════════════════════════════╝
```

---

## Token Estimation Method

AI models do not expose exact token counts in chat interfaces. Use these heuristics:

- 1 token ≈ 4 characters of English text
- 1 token ≈ 0.75 words
- A 200-word message ≈ 267 tokens
- A 100-line Python file ≈ 400-800 tokens depending on line length and comments

Estimate total tokens by summing character counts across all messages and dividing by 4.

Estimate context window size based on known model limits:
- Claude Sonnet / Opus: 200,000 tokens
- GPT-4o: 128,000 tokens
- Llama 3 70B: 8,192 tokens (common default)
- Gemini 1.5 Pro: 1,000,000 tokens

If the model is unknown, default to 8,192 tokens (conservative) and note the assumption.

---

## Top Consumer Analysis

To identify top token consumers:

1. Count characters in all code blocks combined → code tokens
2. Count characters in all AI response text (non-code) → AI response tokens
3. Count characters in all user messages → user message tokens
4. Count characters in all tool call outputs → tool output tokens
5. Show as percentage bar chart (ASCII)

Highlight the top consumer with `[!]` if it represents >40% of total tokens and suggest targeted compression.

---

## Pace Analysis

Calculate:

```
Recent pace = tokens added in last 10 messages
Messages remaining at this pace = (context_remaining) / (recent_pace / 10)
```

Display as:
- "> 20 messages remaining" = green — no urgency
- "10-20 messages" = yellow — consider saving soon
- "< 10 messages" = orange — save recommended
- "< 3 messages" = red — save now, limit imminent

---

## Colour-to-ASCII Mapping (no colour terminal)

| Status | Symbol | Meaning |
|--------|--------|---------|
| Green | `[OK]` | No action needed |
| Yellow | `[~]` | Consider saving soon |
| Orange | `[!]` | Save recommended |
| Red | `[!!]` | Save immediately |
