# The Agentic Setup: No Jargon, Just Context

THERE ARE NO EXPERTS. ONLY THOSE WHO PRETEND TO BE. DO NOT GO PAYING THOUSANDS OF DOLLARS FOR SOMEONE TO TELL YOU WHAT YOU ALREADY KNOW. THIS IS THE AGENTIC SETUP.

---

## The 3 Sections

### 1. Behavior Rules
Controls how the AI communicates and acts. Without it, you get verbose explanations, over-apologizing, and blind execution of risky commands.

### 2. Stack Constraints
Tells the AI exactly what tools, versions, and architecture you're using. Without it, it guesses — and guesses wrong.

### 3. Domain Vocabulary ← most overlooked
Maps your project's terms so the AI doesn't hallucinate naming conventions. If "Client" means something specific in your codebase, say so here. The AI will use it consistently everywhere.

---

## Scope

---
Claude has three scopes for CLAUDE.md — they stack in order:

| Scope | File Location | Loaded When |
|---|---|---|
| User | `~/.claude/CLAUDE.md` | Every session |
| Project | `<repo>/CLAUDE.md` | Inside that repo |
| Sub-folder | `<repo>/src/CLAUDE.md` | Inside that path |

Lower scope wins on conflicts. Put global behavior rules at user scope, project-specific stack/vocab at project scope.


Codex CLI reads `AGENTS.md` before working — same 3-section pattern as Claude. Place it in your repo root. Codex merges files it finds walking up from the current directory, so scope works the same way.

```
~/.codex/AGENTS.md        ← global rules (behavior, style)
<repo>/AGENTS.md          ← project stack + vocab
<repo>/src/AGENTS.md      ← sub-folder overrides
```

To override everything (useful for one-off sessions): `~/.codex/AGENTS.override.md` — this wins over all other files.

---

## Multi-Agent Roles in Claude (and Codex)



### CLAUDE.md 

Define each role as an XML block. Invoke by name in your prompt.

```xml
<agents>
  <swe>
    Scope: implementation only. Write idiomatic, minimal code. No planning output.
    Never expand scope beyond what's asked. Flag blockers, don't work around them.
  </swe>

  <pm>
    Scope: requirements and tradeoffs only. Output tickets or decision summaries, not code.
    Ask clarifying questions before assuming. Think in user outcomes, not technical solutions.
  </pm>

  <qa>
    Scope: adversarial review. Find edge cases, failure modes, and coverage gaps.
    Never defend the implementation — your job is to break it.
    Output: list of test cases and open risks, ranked by severity.
  </qa>
</agents>
```


### Codex 

Same pattern applies. In `AGENTS.md`:

```yaml
agents:
  swe:
    role: Implementation only. Idiomatic code, no scope creep.
  pm:
    role: Requirements and tradeoffs. Output tickets, not code.
  qa:
    role: Adversarial review. Break the implementation. Output ranked risks.
```


**Usage:** Start your message with `[SWE]`, `[PM]`, or `[QA]`

---


## Format by Model

| Model | Preferred Format | Why |
|---|---|---|
| Claude | XML blocks | Trained heavily on XML; creates clear semantic boundaries |
| OpenAI Codex | YAML or JSON | Responds well to key-value structure and schema definitions |
| Gemini | YAML | Human-readable format; XML performs worst |

---

## Templates

### Claude (`CLAUDE.md` or system prompt)

```xml
<behavior_rules>
  - Be terse. No filler. No apologies.
  - Never delete existing comments or error handling unless told to.
  - Confirm before any destructive operation.
</behavior_rules>

<stack>
  - Language: Python 3.11
  - Framework: FastAPI + Pydantic v2
  - Architecture: routes in /routers, DB logic in /crud, models in /models
</stack>

<terminology>
  - "Client" = paying B2B account. Never "Customer" or "User".
  - Transaction states: PENDING | SETTLED | VOIDED only.
  - All PKs: uuid_v7. Timestamps: created_at / updated_at, UTC.
</terminology>
```

---

### OpenAI / Codex (system prompt or workspace file)

```yaml
# WORKSPACE CONTEXT
behavior_rules:
  - Be terse. Prioritize idiomatic code over explanation.
  - Never delete existing comments or error-handling blocks.
  - Verify type safety and null handling before output.

stack:
  language: Python 3.11
  framework: FastAPI / Pydantic v2
  style: PEP 8
  architecture: routes=/routers, db=/crud, models=/models

vocabulary:
  Tenant: Corporate enterprise account, keyed by tenant_id (UUID)
  Seat: Individual license within a Tenant — not a User
```

Schema (when needed):
```json
{
  "Tenant": {
    "tenant_id": "uuid",
    "name": "string",
    "created_at": "datetime"
  },
  "Seat": {
    "seat_id": "uuid",
    "tenant_id": "ref:Tenant",
    "user_email": "email",
    "assigned_at": "datetime"
  }
}
```

---

### Gemini (system prompt)

```yaml
# WORKSPACE CONTEXT
behavior_rules:
  - Be terse. Prioritize clean, idiomatic code.
  - Never delete existing comments or error-handling blocks.
  - Verify bounds, types, and nulls before output.

stack:
  language: Python 3.11
  framework: FastAPI / Pydantic v2
  style: PEP 8
  architecture: routes=/routers, db=/crud, models=/models

vocabulary:
  Tenant: The corporate enterprise account (mapped via tenant_id)
  Seat: An individual license assigned to a user within a Tenant — not the same as a User
```
