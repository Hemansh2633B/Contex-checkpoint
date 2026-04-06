# Cross-Model Compatibility Layer

## Overview

The continuation prompt in Section 6 is always written in a model-agnostic format first. When the user specifies a target model different from the source model, generate an additional model-specific adapter block below the universal prompt.

---

## Universal Prompt Format (always include)

The universal prompt must avoid model-specific phrasing. Rules:

- Do NOT use "As Claude..." or "You are GPT-4..."
- Do NOT reference system prompt conventions of any specific model
- State all context inline (role, constraints, history summary, next task)
- Use plain imperative language: "Read the attached context. Your next task is..."
- Include a model-agnostic style summary (see below)

### Style Summary Block

Capture the conversation's communication style so the target model can match it:

```
### Style Summary (for target model)
Tone: [technical / casual / formal / concise]
User preference: [e.g., "no comments in code", "step-by-step delivery", "direct answers only"]
Reasoning style: [e.g., "show your reasoning before code", "output code first then explain"]
Tool use: [e.g., "user prefers bash over Python for file ops"]
Domain vocabulary: [key terms used in this project — list up to 10]
```

---

## Model-Specific Adapters

Generate the appropriate adapter block based on the user's target model.

### Claude (Anthropic)

```
### Resume Adapter: Claude

<context>
[paste Section 9 Redundancy Block here]
</context>

[paste universal continuation prompt here]
```

Claude responds well to XML context tags and explicit task framing. No special preamble needed.

### GPT-4 / GPT-4o (OpenAI)

```
### Resume Adapter: GPT-4

System: You are a helpful assistant continuing a technical session. The user will paste context from a previous conversation. Read it carefully before responding.

User: [paste universal continuation prompt here]

[paste Section 3 Artifacts here]
```

GPT-4 benefits from an explicit system message framing. Include artifacts inline in the first user message rather than as a second message.

### Gemini (Google)

```
### Resume Adapter: Gemini

[paste universal continuation prompt here]

Context from previous session:
[paste Section 9 Redundancy Block here]
[paste Section 4 Current State here]
```

Gemini handles long context well. Paste the full checkpoint content directly; it does not require special XML tags.

### Llama (local / Ollama / vLLM)

```
### Resume Adapter: Llama

### Instruction
You are continuing a technical session from a saved checkpoint. Do not introduce yourself. Read the context and continue the task.

### Context
[paste Section 2 Key Decisions here]
[paste Section 4 Current State here]

### Task
[paste Section 5 Next Steps item 1 here]

### Input
[paste Section 3 most recent Artifact here]
```

Llama models (instruction-tuned) respond better to explicit `### Section` headers. Keep the injected context short to avoid repetition noise. Use `/resume light` mode for Llama by default.

### Mistral / Mixtral

Use the same adapter as Llama. Mistral instruction models follow the `[INST]...[/INST]` convention natively but accept plain `### Instruction` headers in most frontends.

### Perplexity / Search-augmented models

```
### Resume Adapter: Perplexity

[paste universal continuation prompt here]

Note: This session does not require web search. All context is in the pasted checkpoint. Please do not fetch external URLs unless explicitly instructed.
```

---

## Conflict Resolution During Cross-Model Resume

When a session was saved on one model and resumed on another:

1. The target model may have different default behaviours (verbosity, formatting, tool use).
2. Include the Style Summary block so the target model can match the established tone.
3. If the user reports style mismatch after resume, run `/save` again on the new model — this creates a re-calibrated checkpoint.

---

## Model-Agnostic Reasoning Step Capture

For sessions involving chain-of-thought or multi-step reasoning, include a reasoning transcript summary:

```
### Reasoning Summary (model-agnostic)
Step 1: [what was established]
Step 2: [what was derived from Step 1]
Step 3: [current conclusion or hypothesis]
Open: [what is still uncertain]
```

This allows any model to reconstruct the reasoning chain without re-running it from scratch.
