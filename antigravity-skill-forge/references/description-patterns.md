# Skill Description Patterns

The description is the only part of a skill that's always in context. Everything else loads on demand. This makes the description the single highest-leverage thing you write.

---

## The Three-Part Template

Every Antigravity skill description follows this structure:

```
[CAPABILITY]. Use this skill when [TRIGGER CONTEXTS].
Triggers on: "[phrase 1]", "[phrase 2]", "[phrase 3]", ... .
[SCOPE STATEMENT — what it covers / always-use condition].
```

**Part 1 — Capability statement**
One sentence. Active voice. What does this skill enable Claude to do?

**Part 2 — Trigger contexts**
Complete sentence listing the specific scenarios. Be generous — include adjacent uses.

**Part 3 — Trigger phrase list**
The explicit phrases. These are your keywords. Include the actual words users type, including informal versions.

**Part 4 — Scope / always-use**
When should this skill always fire, even if the user doesn't use an explicit phrase?

---

## Well-Written Descriptions

### mcp-server-builder
```
Build complete, production-ready MCP (Model Context Protocol) servers in
Node.js/TypeScript or Python. Use this skill whenever building an MCP server,
adding MCP tools to an existing service, debugging MCP connectivity, designing
MCP tool schemas, implementing stdio or HTTP transport, or making any service
accessible to AI coding agents like Claude Code or Cursor. Triggers on:
"build an MCP server", "MCP tool", "make this work with Claude Code",
"connect to Cursor", "MCP for my API", "add MCP support",
"Model Context Protocol", "stdio transport", "MCP tool schema",
"expose tools to agents". Always use this skill for any MCP work — it
contains verified patterns for the most common failure modes.
```

What makes it work:
- Capability is specific ("Node.js/TypeScript or Python")
- Trigger contexts cover adjacent uses (debugging, existing service, connectivity)
- Phrase list includes informal versions ("make this work with Claude Code")
- Always-use statement captures cases where user doesn't say "MCP"

### ui-design
```
Create production-grade UI design systems, visual interfaces, and beautiful
component designs. Use this skill whenever the user wants to design a UI,
build a design system, create visual components, style an interface, define
color palettes, typography scales, spacing systems, or produce anything that
requires high-quality visual design decisions. Triggers on: "design this",
"make it look good", "create a UI for", "style this component",
"build a design system", "what should my color palette be",
"how should I structure my typography", "design language", "visual identity",
"design tokens", "dark mode", "light mode", "make this beautiful".
Always use this skill when design quality is explicitly important or when
the user is building something user-facing.
```

What makes it work:
- Covers the full visual design surface area without over-claiming
- Includes natural-language questions ("what should my color palette be")
- Always-use statement is clear and non-overlapping with ux-patterns

---

## Common Description Failures

### Too Short / Too Vague
```
❌ "Helps with API design."
```
This triggers for almost nothing. There are no phrases, no contexts, no scope.

### Too Long / Too Dense
```
❌ "This skill is a comprehensive guide to REST API design covering everything
   from endpoint naming conventions to authentication strategies to rate
   limiting implementations to OpenAPI specification writing to Swagger
   documentation to versioning strategies to deprecation policies to
   response envelope design to pagination patterns to error code taxonomy
   to webhook design to GraphQL alternatives to gRPC considerations..."
```
The trigger mechanism can't extract signal from noise. The description reads like a table of contents, not a trigger condition.

### Overlap With Another Skill
```
❌ ui-design: "...also covers UX patterns and user flow design..."
❌ ux-patterns: "...also includes visual design and color selection..."
```
When two skills overlap in their descriptions, Claude may load both when only one is needed, or choose the wrong one. Keep boundaries clean.

### Missing Trigger Phrases
```
❌ "Use this skill for UI design work."
```
No quoted trigger phrases. Claude needs explicit phrases to pattern-match against user input. Without them, triggering is inconsistent.

### Wrong Voice
```
❌ "This skill will help Claude understand the nuances of..."
✅ "Build production-grade UI design systems..."
```
Write to the task, not about the skill. Start with what gets built, not with the skill's nature.

---

## Boundary Mapping

For every skill, map its neighbours. This prevents overlap and helps write the scope statement.

### Antigravity Stack Boundaries

```
ui-design ↔ ux-patterns
  ui-design owns: how it looks (colors, type, spacing, components visually)
  ux-patterns owns: how it behaves (flows, states, interactions, IA)
  Boundary: "dark mode" → ui-design. "empty state content" → ux-patterns.

ui-design ↔ design-tokens
  ui-design owns: design decisions and philosophy
  design-tokens owns: implementation of those decisions as CSS variables
  Boundary: "what color should my CTA be" → ui-design.
            "how do I structure my colors as CSS variables" → design-tokens.

design-tokens ↔ component-library
  design-tokens owns: the values (--accent, --text-primary)
  component-library owns: using those values in components
  Boundary: "token architecture" → design-tokens.
            "how do I build a Button that uses my tokens" → component-library.

mcp-server-builder ↔ api-designer
  mcp-server-builder owns: MCP protocol, tools, stdio, JSON-RPC
  api-designer owns: REST API design, HTTP endpoints, OpenAPI
  Boundary: "expose this as an MCP tool" → mcp-server-builder.
            "expose this as a REST API" → api-designer.

cli-builder ↔ api-designer
  cli-builder owns: terminal UX, argument parsing, output formatting, distribution
  api-designer owns: HTTP API design
  These rarely conflict — different delivery surfaces.

antigravity-skill-forge ↔ all others
  antigravity-skill-forge owns: building and improving skills themselves
  All others own: their respective domains
  Boundary: always clear — skill-forge is the meta-skill.
```

---

## Testing a Description

After writing a description, test it against 10 prompts: 5 that should trigger, 5 that should not.

**Should trigger — be realistic:**
```
"I need to design a color system for a dark-mode developer tool"      → ui-design ✓
"What's the right typography scale for a B2B SaaS product?"           → ui-design ✓
"Can you help me style these card components?"                        → ui-design ✓
"I want my app to look like Linear — minimal, dark, fast"             → ui-design ✓
"What font should I pair with Cormorant Garamond for a luxury brand?" → ui-design ✓
```

**Should not trigger:**
```
"How should my onboarding flow work for new users?"  → ux-patterns, not ui-design
"Should this button be a modal or a new page?"       → ux-patterns, not ui-design
"How do I structure my CSS variables?"               → design-tokens, not ui-design
"Build a reusable Button component in React"         → component-library, not ui-design
"What's the best font for code?"                     → generic knowledge, no skill
```

If the description is triggering the wrong skill, check: are the boundary conditions explicit in both descriptions?