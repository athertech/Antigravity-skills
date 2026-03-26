# Antigravity Skill Inventory

Complete record of all skills in the Antigravity stack. Update this file every time a skill is added, improved, or retired.

Last updated: March 2026

---

## Active Skills

### Design & UI

#### ui-design
- **Job:** Visual design systems — palettes, typography, spacing, component aesthetics, dark mode
- **Owns:** How things look
- **Does NOT own:** How things behave (→ ux-patterns), CSS variable implementation (→ design-tokens), React component code (→ component-library)
- **Key content:** Color architecture (primitive→semantic), typography scale/pairing, spacing system, dark mode implementation, design review checklist
- **Trigger phrases:** "design this", "color palette", "make it look good", "typography", "dark mode", "visual identity", "design system"

#### ux-patterns
- **Job:** Interaction design, user flows, navigation IA, all UI states
- **Owns:** How things behave and flow
- **Does NOT own:** How things look (→ ui-design), component code (→ component-library)
- **Key content:** Flow mapping methodology, onboarding patterns, navigation selection, 7 UI states (empty/loading/error/etc.), form UX, notification patterns, accessibility UX
- **Trigger phrases:** "user flow", "onboarding", "empty state", "error state", "navigation", "UX for", "information architecture", "how should this work"

#### design-tokens
- **Job:** Systematic design token architecture — primitives, semantics, CSS custom properties, multi-theme, export formats
- **Owns:** Design value implementation and organisation
- **Does NOT own:** Design decisions (→ ui-design), component usage of tokens (→ component-library)
- **Key content:** Primitive→semantic layering, complete CSS variable implementations (light+dark), W3C token JSON format, Style Dictionary config, multi-brand architecture
- **Trigger phrases:** "design tokens", "CSS variables", "CSS custom properties", "token architecture", "theme system", "multi-theme", "Figma tokens", "W3C design tokens"

#### component-library
- **Job:** Reusable UI component APIs and implementations — Button, Input, Card, Modal, Badge
- **Owns:** Component code, prop APIs, accessibility implementation, component folder structure
- **Does NOT own:** Visual design decisions (→ ui-design), token architecture (→ design-tokens)
- **Key content:** Component API design principles, full TypeScript+accessibility implementations, ARIA requirements by component, cn() utility, folder structure
- **Trigger phrases:** "component library", "design system", "reusable component", "Button component", "Input component", "Modal component", "compound components", "accessible component", "headless component"

---

### Developer Tooling

#### mcp-server-builder
- **Job:** MCP (Model Context Protocol) server implementation in TypeScript or Python
- **Owns:** MCP protocol, tool schemas, stdio transport, JSON-RPC, HTTP transport for remote servers
- **Does NOT own:** REST API design (→ api-designer), CLI design (→ cli-builder)
- **Key content:** Critical stdout/stderr rule, transport selection, TypeScript skeleton, Python/FastMCP skeleton, tool description writing guide, error patterns, MCP Inspector testing, HTTP deployment
- **Trigger phrases:** "MCP server", "MCP tool", "Model Context Protocol", "Claude Code integration", "Cursor integration", "stdio transport", "expose tools to agents"
- **Critical rule:** console.log() corrupts stdio stream — document in every session

#### cli-builder
- **Job:** Production CLI tools in Node.js (Commander.js) or Python (Typer)
- **Owns:** Command structure, argument parsing, output formatting, config discovery, interactive prompts, distribution
- **Does NOT own:** API design (→ api-designer), MCP tool design (→ mcp-server-builder)
- **Key content:** Commander.js and Typer patterns, output formatting (color/spinners), config discovery priority, interactive prompts, exit codes, npm/pip/Homebrew distribution, CLI quality checklist
- **Trigger phrases:** "build a CLI", "command line tool", "terminal tool", "npm package CLI", "Commander", "Typer", "Click", "argparse", "subcommands"

#### api-designer
- **Job:** REST API design and Express implementation
- **Owns:** URL structure, HTTP semantics, response schemas, error design, authentication, rate limiting, OpenAPI
- **Does NOT own:** MCP tools (→ mcp-server-builder), CLI commands (→ cli-builder)
- **Key content:** URL structure rules, HTTP method semantics, response envelopes, error codes, API key auth pattern, SQLite rate limiter, Express skeleton, OpenAPI 3.1 template, API quality checklist
- **Trigger phrases:** "design an API", "REST API", "API endpoints", "OpenAPI", "Swagger", "Express API", "Fastify", "Hono", "API authentication", "rate limiting", "API versioning"

---

### Meta

#### antigravity-skill-forge
- **Job:** Building, improving, and maintaining skills themselves
- **Owns:** Skill architecture decisions, description writing, quality standards, the Antigravity skill stack itself
- **Does NOT own:** Any specific domain skill (those are the other skills)
- **Key content:** What makes a skill worth building, two-layer rule (declarative/imperative), description trigger mechanism, build process phases, quality standards, anti-patterns, improvement protocol
- **Reference files:** structures.md, description-patterns.md, test-prompts.md, skill-inventory.md (this file)
- **Trigger phrases:** "build a skill", "create a skill", "skill for X", "SKILL.md", "skill architecture", "skill description", "improve this skill"

---

## Planned Skills (Not Yet Built)

| Skill Name | Job | Priority |
|---|---|---|
| `db-schema` | Database schema design, migrations, indexing patterns | High |
| `auth-patterns` | Authentication flows, JWT, OAuth, session design | High |
| `testing-patterns` | Test strategy, unit/integration/e2e patterns, coverage | Medium |
| `deployment` | Railway, Vercel, Fly.io, Docker deployment patterns | Medium |
| `react-state` | State management selection and patterns (Zustand, Redux, Context) | Medium |
| `performance` | Web performance, Core Web Vitals, profiling | Low |

---

## Retired Skills

None yet.

---

## Skill Versioning

When a skill is meaningfully updated, log it here:

| Skill | Version | Change | Date |
|---|---|---|---|
| All skills | 1.0 | Initial build | March 2026 |