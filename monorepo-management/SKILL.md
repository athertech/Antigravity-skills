---
name: monorepo-management
description: Architect and manage high-scale monorepos for complex web applications and design systems. Use this skill when setting up Turborepo or Nx workspaces, organizing shared packages (UI, utils, config), managing cross-workspace dependencies, optimizing CI/CD with remote caching, implementing automated versioning (Changesets), or designing a multi-entry-point library. Triggers on: "monorepo setup", "Turborepo", "Nx", "shared packages", "workspace management", "monorepo architecture", "CI/CD caching", "pnpm workspaces", "npm workspaces", "Yarn workspaces", "Lerna", "Changesets", "package-based monorepo". Always use this skill for any project with multiple apps or packages.
---

# Monorepo Management Skill

Build and scale a unified codebase for multiple applications and shared libraries. This skill focuses on the architectural decisions and tooling for Turborepo and pnpm workspaces.

---

## Workspace Architecture: The "Shared First" Pattern

A successful monorepo is built on a clear package hierarchy:

- **apps/**: Deployable units (Web, API, CLI, Admin).
- **packages/**: Internal libraries (UI System, Utils, Config).
- **tooling/**: Shared CI/CD scripts, ESLint/Prettier configs, and database schemas.

---

## Implementation: pnpm Workspaces (Recommended)

pnpm is the standard for fast, space-efficient monorepos.

### 1. `pnpm-workspace.yaml`

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
  - 'tooling/*'
```

### 2. Dependency Management

Always link local packages using `workspace:*`.

```json
// apps/web/package.json
{
  "dependencies": {
    "next": "14.2.0",
    "@acme/ui": "workspace:*",
    "@acme/utils": "workspace:*"
  }
}
```

---

## Tooling: Turborepo (Recommended for Build Orchestration)

Turborepo handles task scheduling and caching.

### 1. `turbo.json` — The Pipeline Pattern

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"], // Build dependencies first
      "outputs": [".next/**", "dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    },
    "lint": {},
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### 2. Remote Caching in CI (GitHub Actions)

Use Turborepo's caching to reduce build times by up to 90%.

```yaml
- name: Cache Turbo
  uses: actions/cache@v3
  with:
    path: .turbo
    key: ${{ runner.os }}-turbo-${{ github.sha }}
    restore-keys: |
      ${{ runner.os }}-turbo-
```

---

## Designing Shared Packages

Avoid the "Big Shared Package" antipattern. Instead, use specialized packages.

### 1. The "@acme/ui" Pattern
- **Export**: Individual components.
- **Transpile**: Let the consumer (app) handle transpilation via `transpilePackages` in Next.js.
- **Style**: Use a shared Tailwind config at the root.

### 2. The "@acme/config" Pattern
Pasting ESLint and TypeScript configs once at the root and extending them.

```json
// packages/typescript-config/base.json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "NodeNext",
    "strict": true,
    "skipLibCheck": true
  }
}
```

---

## Versioning & Publishing (Changesets)

Use `Changesets` to manage versioning across multiple packages.

1. **Add a change**: `pnpm changeset` (Interactively select packages and impact).
2. **Version packages**: `pnpm changeset version` (Updates all `package.json` files).
3. **Publish**: `pnpm changeset publish` (Publishes only changed packages to npm).

---

## Monorepo Quality Checklist

- [ ] **Lockfile**: Unified lockfile at the root. No nested lockfiles.
- [ ] **Dependencies**: No duplicate versions of React/Tailwind across packages.
- [ ] **Scripts**: All apps/packages use consistent lifecycle scripts (`build`, `test`, `lint`).
- [ ] **Pruning**: Using `turbo prune` for Docker builds to keep images small.
- [ ] **Visibility**: No direct cross-imports between `apps/`. Always use a `packages/` bridge.
- [ ] **CI**: Pull Requests only run tests for affected packages (`turbo test --filter=...[origin/main]`).

---

## Reference Resources

- [Turborepo Documentation](https://turbo.build/repo/docs)
- [pnpm Workspaces Guide](https://pnpm.io/workspaces)
- [Monorepo.tools](https://monorepo.tools/)
