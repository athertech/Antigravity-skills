# Skill Structures — Decision Tree

## Choosing a Structure

Work through these questions in order. Stop at the first match.

---

### Does the skill need to run commands or execute scripts?

**Yes →** Multi-file skill with `scripts/`

Examples: `docx`, `xlsx`, `web-artifacts-builder`

Structure:
```
skill-name/
├── SKILL.md          — Workflow, when to load each script
├── scripts/
│   ├── build.sh      — Executable scripts
│   └── validate.py
└── references/
    └── schema.md     — Supporting reference data
```

Rules for scripted skills:
- Every script must have a clear, single purpose
- SKILL.md explains when and why to run each script, not what it does internally
- Scripts handle the deterministic work; SKILL.md handles the judgment work

---

### Does the skill serve multiple distinct domains or tech stacks?

**Yes →** Multi-domain skill with `references/{domain}.md`

Examples: `mcp-server-builder` (TypeScript vs Python), a hypothetical `deploy` skill (AWS vs GCP vs Railway)

Structure:
```
skill-name/
├── SKILL.md          — Selection logic: how to pick the right domain
└── references/
    ├── typescript.md — TypeScript-specific patterns
    ├── python.md     — Python-specific patterns
    └── shared.md     — Common patterns across domains
```

Rules for multi-domain skills:
- SKILL.md should read like a decision tree, not like documentation
- Each domain reference file is self-contained — Claude loads only one
- Shared concepts live in shared.md and are referenced by both domain files

---

### Does the skill have large reference data (schemas, tables, examples over 50 lines)?

**Yes →** Single SKILL.md + `references/`

Examples: `ui-design`, `api-designer`, `component-library`

Structure:
```
skill-name/
├── SKILL.md          — Framework, principles, key patterns
└── references/
    ├── palettes.md   — The big lookup tables
    ├── components.md — Full implementations
    └── checklist.md  — Review criteria
```

Rules:
- SKILL.md contains the decision-making framework and short illustrative examples
- Reference files contain the full content that SKILL.md points to
- Each reference file has a ToC if it exceeds 100 lines

---

### Is this a focused skill with no external data needs?

**Yes →** Single SKILL.md only

Examples: `frontend-design`, `ux-patterns`, `brand-guidelines`

Structure:
```
skill-name/
└── SKILL.md          — Everything, under 300 lines ideally
```

Rules:
- Under 300 lines is ideal
- Under 500 lines is the hard limit
- If you're approaching 500, you're embedding reference data — extract it

---

## Structure Anti-Patterns

**Everything in SKILL.md** — The most common mistake. 600-line SKILL.md files with full schemas, complete implementations, and lookup tables inline. Split aggressively.

**Too many reference files** — If you have 8 reference files, your skill might need to be split into 2–3 narrower skills. Reference files should feel like chapters, not encyclopaedias.

**Scripts for things that should be in SKILL.md** — If the script is just `echo "here's a template"`, don't make it a script. Inline it or put it in a reference file.

**Nested references** — A reference file that points to another reference file. Load depth max 2 (SKILL.md → one reference file). Don't chain.

---

## File Naming

```
SKILL.md                  — Always this exact name, capitalised
scripts/                  — Executable code
references/               — Non-executable content (md, json, yaml)
assets/                   — Static files used in output (templates, fonts, images)

Reference file names:
  snake_case.md           — All lowercase, underscores
  Be specific:
    color-palettes.md     ✓
    content.md            ✗ (too vague)
    reference-01.md       ✗ (use the actual content name)
```