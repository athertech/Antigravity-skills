---
name: antigravity-skill-forge
description: Antigravity's master skill for designing, building, testing, and shipping production-ready skills for any platform or agent. Use this skill whenever building a new skill from scratch, improving an existing skill, designing the architecture of a skill system, writing skill descriptions that trigger reliably, deciding what belongs in a skill vs. in code, structuring multi-file skills with references, creating skills for UI design, UX, design tokens, MCP servers, CLIs, APIs, or component libraries. Triggers on: "build a skill", "create a skill for", "make a skill that", "turn this into a skill", "skill for X", "how should I structure this skill", "write the SKILL.md", "what should go in the skill description", "skill architecture". Always use this skill when Antigravity is building, improving, or thinking about skills — it is the meta-skill that governs how all other skills are made.
---

# Antigravity Skill Forge

This is the master skill for building skills at Antigravity. It captures the philosophy, process, architecture decisions, and quality standards that make a skill genuinely useful — not just syntactically correct, but something that makes the coding agent measurably better at a specific job.

Read this skill fully before starting. The order matters.

---

## What Makes a Skill Worth Building

A skill is worth building when it meets one of these three criteria:

**1. Repetitive expertise** — You find yourself giving the same detailed instructions across multiple sessions. The skill captures that expertise once and applies it every time.

**2. Tribal knowledge** — There's something your team knows that Claude doesn't: your stack, your patterns, your standards, your constraints. The skill closes that gap.

**3. Quality floor** — Without a skill, the output is acceptable. With a skill, it's distinctly better — not because Claude couldn't figure it out, but because the skill saves the 10 minutes of context-setting that would have produced the same result anyway.

If none of these three are true, write a good prompt instead. Skills are for recurring, expert-level workflows. They are not documentation.

---

## Skill Architecture Principles

### The Two-Layer Rule

Every good skill has two layers. Understand which layer you're writing before you write anything.

**Layer 1 — Declarative (the SKILL.md body)**
The philosophy, the decision framework, the "why". This is read and reasoned over. It produces judgment.

**Layer 2 — Imperative (scripts/, references/)**
The exact commands, the precise schema, the copy-paste implementation. This is executed. It produces consistency.

The mistake most skill authors make: they write Layer 2 content in Layer 1. They put a 400-line JSON schema in SKILL.md when it belongs in `references/schema.json`. SKILL.md should tell Claude *how to think*. The reference files tell Claude *exactly what to do*.

### Scope: One Job, Done Completely

A skill should have one clear job. If you're describing what the skill does and you use the word "and" more than twice, split it.

```
❌ "This skill handles API design and helps with database schemas and also covers authentication."

✅ api-designer.skill    — REST API design, endpoint structure, OpenAPI
✅ db-schema.skill       — Schema design, migrations, indexing
✅ auth-patterns.skill   — Authentication flows, JWT, session design
```

Narrow scope doesn't mean thin content. A skill with one job can be deep and thorough within that job. Breadth is the enemy of quality.

### The Description Is the Trigger Mechanism

The description field is not documentation. It is the signal Claude uses to decide whether to load the skill. Write it as if you're writing trigger conditions, not a product description.

Three parts, always in this order:

```
1. What the skill does           (one sentence, capability statement)
2. When to use it                (explicit trigger phrases and contexts)
3. What it covers                (scope statement to prevent over-triggering)
```

Make it "pushy" — err toward triggering too often rather than too rarely. An under-triggered skill that would have helped is a missed opportunity. An over-triggered skill that didn't help costs one extra context load.

```
❌ "A skill for designing APIs."

✅ "Design and build production-quality REST APIs with consistent patterns,
   proper error handling, authentication, and rate limiting. Use this skill
   when designing API endpoints, structuring API responses, writing OpenAPI
   specs, building Express/Fastify/Hono APIs, or reviewing existing API
   design. Triggers on: 'design an API', 'REST API', 'API endpoints',
   'OpenAPI', 'Swagger', 'Express API', 'API authentication', 'rate limiting'.
   Always use when API design consistency and DX matter."
```

---

## The Antigravity Skill Stack

These are the skills Antigravity has built. Know them. Reference them. Don't duplicate them.

| Skill | Job | Triggers On |
|---|---|---|
| `ui-design` | Visual design systems, palettes, typography, dark mode | "design this", "color palette", "make it look good" |
| `ux-patterns` | User flows, states, navigation IA, interaction design | "how should this flow", "empty state", "onboarding" |
| `design-tokens` | CSS custom properties, token architecture, W3C format | "design tokens", "CSS variables", "theme system" |
| `component-library` | Reusable React/Vue components with accessible APIs | "component library", "button component", "modal" |
| `mcp-server-builder` | Production MCP servers in TypeScript or Python | "build an MCP", "MCP tool", "Claude Code integration" |
| `cli-builder` | Production CLIs via Commander.js or Typer | "build a CLI", "command line tool", "npm package" |
| `api-designer` | REST API design, Express structure, OpenAPI | "design an API", "REST API", "rate limiting" |
| `antigravity-skill-forge` | Building and improving skills themselves | "build a skill", "skill for X", "SKILL.md" |

Before building a new skill, check: does this job already exist in the stack? If so, improve the existing skill rather than creating a duplicate.

---

## Build Process

### Phase 1: Capture Intent

Start here. Do not skip to writing SKILL.md.

Answer these four questions before touching a file:

**What job does this skill do?**
Write one sentence. If you can't write one sentence, the scope isn't defined yet.

**When does it trigger?**
List 5–10 specific phrases a user would actually type. Not abstract — real phrases. "Can you help me with design" is abstract. "What font should I pair with Playfair Display for a fintech product" is real.

**What does it NOT cover?**
The boundary conditions prevent the skill from bloating. Name the adjacent jobs that belong in a different skill.

**What does success look like?**
What is measurably different about Claude's output when this skill is loaded vs. when it isn't? If the answer is "not much", the skill isn't worth building.

### Phase 2: Decide the Structure

Read `references/structures.md` for the decision tree. The short version:

```
Simple skill (one job, no tooling)
→ Single SKILL.md, under 300 lines
→ Examples: ui-design, ux-patterns

Skill with reference data (schemas, checklists, large tables)
→ SKILL.md (the framework) + references/ (the data)
→ Examples: mcp-server-builder, api-designer

Skill with executable tooling (scripts that run)
→ SKILL.md + scripts/ + references/
→ Examples: web-artifacts-builder, docx, xlsx

Multi-domain skill (same job, different contexts)
→ SKILL.md (selection logic) + references/{domain}.md
→ Example: deploy-skill with references/aws.md, references/gcp.md
```

### Phase 3: Write SKILL.md

Write the body in this order:

1. **Opening statement** — One paragraph. What problem does this skill solve and why does it matter? No headers, no bullets. Just a crisp statement of purpose.

2. **Decision framework** — The mental model. How should Claude think about this domain? What are the key decisions, and how does one choose? This is the "thinking" layer.

3. **Patterns and implementations** — The specific patterns, with real code, real examples, real schema. Reference files for anything over ~50 lines.

4. **Quality checklist** — The criteria Claude uses to verify its own output. Numbered, objective, checkable.

5. **Reference file pointers** — What's in the reference files and when to load them.

Keep SKILL.md under 500 lines. Under 300 is better. If you're approaching 500, you're embedding reference content in the skill body. Move it to `references/`.

### Phase 4: Write Reference Files

Reference files are loaded on demand — they don't add to the context unless Claude needs them. This makes them free to be comprehensive. Put in them:

- Complete code implementations (over ~30 lines)
- Full schemas and type definitions
- Lookup tables (color palettes, font pairings, pattern libraries)
- Domain-specific deep dives that only apply to some invocations
- Examples that are illustrative but not always needed

Name reference files by what they contain, not by number:
```
references/
├── color-palettes.md      ← Not references/01.md
├── type-pairings.md
├── component-specs.md
└── accessibility.md
```

### Phase 5: Write the Description

Write it last. You now know what the skill actually does — the description you wrote in Phase 1 was a hypothesis. Rewrite it based on what you built.

Use the three-part structure from above: capability, trigger phrases, scope.

Run it through one test: read the description and ask "would a slightly obtuse version of Claude load this skill for a user typing [real example prompt]?" If the answer is no, make the description more explicit.

---

## Skill Quality Standards

These are Antigravity's minimum bar for shipping a skill.

### Does it actually improve output?

Load the skill and run the three canonical test prompts from `references/test-prompts.md` for that domain. Without the skill, what does Claude produce? With the skill, what does Claude produce? Is the difference meaningful and consistent?

If you can't tell the difference in a blind review, the skill isn't working. Either the content is too thin, the description isn't triggering, or the job was too easy to do without a skill.

### Does it explain the why?

Every instruction in SKILL.md should have a reason — either explicit ("Use `retireBrowserAfterPageCount: 50` because Playwright accumulates renderer state") or implied by context. Instructions without reasons produce rigid behavior. Reasons produce judgment.

If you find yourself writing "ALWAYS" or "NEVER" in all-caps, stop and ask: what's the reason this must always or never happen? Write the reason. The all-caps command becomes unnecessary.

### Is it scoped correctly?

A skill is correctly scoped when:
- You can describe its job in one sentence without "and"
- Its SKILL.md doesn't contain content that should be in a different skill
- Its description doesn't compete with another skill's description

### Is the description triggering correctly?

Test trigger fidelity with five prompts that should trigger and five that should not. The skill should load for all five trigger prompts and none of the five non-trigger prompts.

Common trigger failures:
- Description too abstract (skill loads for nothing)
- Description too narrow (skill loads only for exact phrasing)
- Description overlaps another skill (both load when only one should)

---

## Skill Improvement Protocol

When improving an existing skill rather than building from scratch:

**Step 1: Diagnose first.** Run the existing skill against 3 real prompts. What specifically is wrong with the output? Is it the content of the skill (wrong advice), the structure (right advice, can't find it), or the trigger (right advice, skill not loading)?

**Step 2: Make one change at a time.** Improving a skill by making 5 changes simultaneously makes it impossible to know which change helped. Pick the most important improvement and test it.

**Step 3: Preserve the voice.** The existing skill has a tone and approach that may have been deliberate. If it's working for most cases, preserve it and add to it rather than rewriting.

**Step 4: Don't let SKILL.md grow indefinitely.** Every improvement that adds 10 lines should remove 5. Skills that grow without pruning become difficult to follow and trigger unreliably.

**Step 5: Update the description last.** Same as building from scratch — wait until you know what the skill actually does, then rewrite the description to match.

---

## Antipatterns to Avoid

**The Instruction Dump** — Pasting documentation into SKILL.md. Skills are not README files. They are reasoning frameworks. If it reads like a reference manual, it belongs in `references/`, not in the body.

**The Micro-Skill** — A skill that does something so simple Claude would do it correctly without a skill. "How to name variables" is not a skill. Save skills for jobs where domain knowledge matters.

**The Everything Skill** — A skill that covers 5 different jobs because they're vaguely related. This produces bad triggering and diluted content. Split it.

**The Stale Skill** — A skill that was correct when written but hasn't been updated as the stack evolved. Review skills whenever a dependency or pattern changes. A skill giving wrong advice is worse than no skill.

**The MUST-Obsessed Skill** — A skill full of "MUST always", "NEVER do this", "YOU MUST". Heavy-handed commands produce compliant behavior, not intelligent behavior. Explain the why. Trust the model.

---

## Reference Files

Load these when you need them:

- `references/structures.md` — Decision tree for choosing skill structure (single file vs multi-file vs scripted)
- `references/description-patterns.md` — Templates and anti-patterns for skill descriptions
- `references/test-prompts.md` — Canonical test prompts for each skill in the Antigravity stack
- `references/skill-inventory.md` — Full inventory of all Antigravity skills, their jobs, and their boundaries