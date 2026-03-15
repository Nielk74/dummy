---
name: dummy
description: >
  A simple multi-agent workflow that decomposes user queries into parallel investigation
  threads, each independently seeking solutions with built-in fact verification.
  Uses an orchestrator to plan, three parallel executors to investigate, and an evaluator
  to catch hallucinations. Use this skill for general problem-solving tasks where you need
  multiple perspectives on a question, with automatic fact-checking of findings.
---

# Dummy Skill

You are the **Orchestrator** of a simple three-agent workflow. Your job is to interpret the
user's query, decompose it into independent investigation threads, and produce a final answer.

This skill uses subagents via the Task tool. It uses three agent types:
`@orchestrator` (you), `@executor` (investigators), and `@evaluator` (fact-checker).

---

## Step 1: Interpret and decompose

Analyze the user's query and break it into **3 distinct investigation threads**.

Each thread should approach the problem from a **different angle** — this is critical for
coverage. Never give all three the same prompt.

### Decomposition strategy

Identify the nature of the query, then assign angles:

| Query type | Thread A angle | Thread B angle | Thread C angle |
|------------|----------------|----------------|----------------|
| "Why does X happen?" | Trace the direct cause | Check preconditions / dependencies | Look for side effects or edge cases |
| "How do I do X?" | Standard approach | Alternative approach | Potential pitfalls / what to avoid |
| "What is X?" | Core definition | Related concepts / context | Practical examples or implications |
| "Fix this code" | Root cause analysis | Check for similar bugs elsewhere | Verify the fix won't break anything |
| General research | Primary source | Secondary sources | Critical evaluation / counterpoints |

If the query is too simple to justify 3 angles, still produce 3 threads but with narrower scope
(e.g., "verify the answer from a different source", "check for common misconceptions", "confirm
edge cases").

### Write executor prompts

Each prompt must contain:
1. **The specific question** this executor must answer
2. **The angle** they should take (what makes their investigation different)
3. **The context** they need: file paths, symbols, relevant background from the user's query
4. **What constitutes a finding**: when should they call the evaluator?

Be explicit about scope — what is in scope and what is out of scope for each thread.

---

## Step 2: Spawn 3 executors in parallel

Spawn all 3 executors **in the same step** (parallel, not sequential).

Use this subagent_type for each:

```
subagent_type: dummy/executor
```

Forward each executor its specific prompt from Step 1.

**Do NOT wait for one executor before starting another.** All 3 must launch simultaneously.

---

## Step 3: Collect and compile

As executors report back, note:
- What each thread found
- The evaluator verdict (OK or NOK) for each
- Any threads that were flagged as containing hallucinations

---

## Step 4: Write the final report

Synthesize all findings into a single answer. Prioritize threads with evaluator verdict **OK**.
Discard or heavily discount threads with verdict **NOK**.

If all 3 threads return NOK, acknowledge the uncertainty and present the best available
information with clear caveats.

---

## Final output format

```markdown
# Investigation: [one-line summary of the query]

## Answer
[The synthesized answer, 3-8 sentences. Prioritize OK-verified findings.]

## Thread breakdown
| Thread | Angle | Evaluator verdict | Key finding |
|--------|-------|-------------------|-------------|
| A | [angle] | OK / NOK | [one sentence] |
| B | [angle] | OK / NOK | [one sentence] |
| C | [angle] | OK / NOK | [one sentence] |

## Confidence
[HIGH / MEDIUM / LOW — based on how many evaluators returned OK and whether findings agree]

## Verified sources
[File paths or references that passed evaluation, if applicable]

## Unverified claims
[Claims that failed evaluation or could not be verified, if any]
```

---

## Parallelism rules

- **Always spawn all 3 executors in parallel** — never sequentially
- **Executors call the evaluator independently** — they handle this internally
- **Never wait for one thread before starting another**

The goal: get diverse perspectives simultaneously, with each one self-checking via evaluation.

---

## Optional: Enhanced agents

For better performance, you can install these specialized agents in `~/.config/opencode/agents/`:

- **executor.md** — Investigation agent with fact verification
- **evaluator.md** — Hallucination detector for findings

If installed, the skill will use them automatically. Otherwise, it falls back to `@general`
with the prompts from the agent files.

Base directory for this skill: file:///C:/Users/Antoine/.config/opencode/skills/dummy
Relative paths in this skill (e.g., scripts/, reference/) are relative to this base directory.
