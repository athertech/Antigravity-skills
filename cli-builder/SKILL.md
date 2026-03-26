---
name: cli-builder
description: Build production-quality CLI tools in Node.js or Python. Use this skill whenever building a command-line interface, adding CLI commands to an existing project, designing CLI argument schemas, implementing subcommands, adding interactive prompts, building a developer tool distributed via npm or pip, or wrapping an API with a terminal interface. Triggers on: "build a CLI", "command line tool", "add a CLI", "npm package CLI", "terminal tool", "argparse", "commander", "yargs", "click", "typer", "subcommands", "interactive prompts", "CLI for my API", "distribute as npm package". Always use this skill for any CLI work — it contains production patterns for argument parsing, output formatting, error handling, and distribution.
---

# CLI Builder Skill

Build production-quality CLIs: well-structured commands, great output formatting, smart error handling, and smooth distribution. Works for Node.js (Commander.js) and Python (Typer / Click).

## CLI Design Principles

Before writing code, decide on these five things:

**1. Command structure**
```
tool <command> [subcommand] [arguments] [options]

# Examples:
uiskill analyze https://stripe.com
uiskill skills list --tags dark-mode
uiskill export linear.app --format json --output ./skills/
gh pr create --title "Fix bug" --body "..."
```

**2. Output contract**
- Default output: human-readable, formatted for terminal
- `--json` flag: machine-parseable JSON for piping
- `--quiet` flag: suppress non-essential output (good for CI)
- `--verbose` flag: extra debug info

**3. Exit codes**
```
0   — Success
1   — General error
2   — CLI usage error (wrong arguments)
3+  — Domain-specific errors (e.g. 3 = not found, 4 = auth error)
```

**4. Config discovery**
```
Priority (highest to lowest):
  CLI flags         --api-key=xxx
  Environment vars  UISKILL_API_KEY=xxx
  Config file       ~/.uiskill/config.json
  Defaults
```

**5. Interactive vs scriptable**
If the terminal is a TTY (interactive), show spinners, colors, prompts.
If stdin/stdout is piped (scripting), suppress color, suppress spinners, output raw data.

```javascript
const isInteractive = process.stdout.isTTY;
```

---

## Node.js CLI (Commander.js)

### Project Structure

```
your-cli/
├── package.json
├── src/
│   ├── cli.js          — Entry point, program setup
│   ├── commands/
│   │   ├── analyze.js  — One file per command
│   │   ├── skills.js
│   │   └── export.js
│   ├── config.js       — Config file loading
│   ├── output.js       — Formatters, colors, spinners
│   └── api.js          — API client
└── bin/
    └── uiskill         — Symlink or thin wrapper
```

### package.json

```json
{
  "name": "uiskill",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "uiskill": "./bin/uiskill.js"
  },
  "scripts": {
    "start": "node src/cli.js",
    "test": "node --test tests/"
  },
  "dependencies": {
    "commander": "^12.0.0",
    "ora": "^8.0.0",
    "chalk": "^5.3.0",
    "conf": "^13.0.0",
    "prompts": "^2.4.2"
  }
}
```

### bin/uiskill.js

```javascript
#!/usr/bin/env node
import '../src/cli.js';
```

### src/cli.js — Main entry

```javascript
import { program } from 'commander';
import { analyzeCommand } from './commands/analyze.js';
import { skillsCommand } from './commands/skills.js';
import { exportCommand } from './commands/export.js';
import { pkg } from './config.js';

program
  .name('uiskill')
  .description('UI intelligence platform CLI')
  .version(pkg.version)
  .option('--json', 'Output as JSON (machine-readable)')
  .option('--quiet', 'Suppress non-essential output')
  .option('--api-key <key>', 'API key (overrides UISKILL_API_KEY env var)');

program.addCommand(analyzeCommand);
program.addCommand(skillsCommand);
program.addCommand(exportCommand);

// Global error handler
program.exitOverride();

try {
  await program.parseAsync(process.argv);
} catch (err) {
  if (err.code === 'commander.unknownCommand') {
    console.error(`Unknown command: ${err.message}`);
    program.help();
    process.exit(2);
  }
  console.error(`Error: ${err.message}`);
  process.exit(1);
}
```

### src/commands/analyze.js — Subcommand pattern

```javascript
import { Command } from 'commander';
import ora from 'ora';
import { createApiClient } from '../api.js';
import { formatSkill, printError, printSuccess } from '../output.js';

export const analyzeCommand = new Command('analyze')
  .description('Analyze a live website and return its UI Skill')
  .argument('<url>', 'URL to analyze (include https://)')
  .option('-t, --page-type <type>', 'Page type: homepage|pricing|docs|landing|dashboard', 'homepage')
  .option('-p, --provider <provider>', 'LLM provider: claude|openai|gemini', 'claude')
  .option('-o, --output <path>', 'Save skill JSON to file')
  .action(async (url, opts, cmd) => {
    const globalOpts = cmd.parent.opts();
    const api = createApiClient(globalOpts.apiKey);

    // Validate URL
    try { new URL(url); } catch {
      printError('Invalid URL. Include https:// e.g. https://stripe.com');
      process.exit(2);
    }

    const spinner = globalOpts.quiet ? null : ora(`Analyzing ${url}...`).start();

    try {
      const skill = await api.analyze(url, { pageType: opts.pageType, provider: opts.provider });

      spinner?.succeed(`Analyzed ${skill.domain}`);

      if (globalOpts.json || !process.stdout.isTTY) {
        // Machine-readable output
        console.log(JSON.stringify(skill, null, 2));
      } else {
        // Human-readable output
        formatSkill(skill);
      }

      if (opts.output) {
        const fs = await import('fs/promises');
        await fs.writeFile(opts.output, JSON.stringify(skill, null, 2));
        printSuccess(`Saved to ${opts.output}`);
      }

    } catch (err) {
      spinner?.fail('Analysis failed');
      printError(err.message);
      process.exit(err.exitCode ?? 1);
    }
  });
```

### src/output.js — Formatting and color

```javascript
import chalk from 'chalk';

// Detect if color is supported
const useColor = process.stdout.isTTY && !process.env.NO_COLOR;
const c = useColor ? chalk : { bold: s=>s, dim: s=>s, green: s=>s, red: s=>s, blue: s=>s, gray: s=>s };

export function printSuccess(msg) {
  console.log(`${c.green('✓')} ${msg}`);
}

export function printError(msg) {
  console.error(`${c.red('✗')} ${msg}`);
}

export function printWarning(msg) {
  console.warn(`${c.yellow('!')} ${msg}`);
}

export function formatSkill(skill) {
  console.log();
  console.log(c.bold(`${skill.domain}`), c.dim(`· ${skill.niche}`));
  console.log(c.dim('─'.repeat(50)));

  console.log(c.bold('Design principles:'));
  skill.design_principles.forEach(p => console.log(`  ${c.dim('·')} ${p}`));

  console.log();
  console.log(c.bold('Typography:'), skill.typography.primary_font, c.dim(`· ${skill.typography.signal}`));
  console.log(c.bold('Accent:'), skill.color_language.accent);
  console.log(c.bold('Tags:'), skill.tags.map(t => c.blue(t)).join(' '));

  if (skill.meta?.confidence) {
    const conf = Math.round(skill.meta.confidence * 100);
    const confColor = conf > 75 ? c.green : conf > 50 ? c.yellow : c.red;
    console.log(c.bold('Confidence:'), confColor(`${conf}%`));
  }
  console.log();
}
```

### src/config.js — Config loading

```javascript
import Conf from 'conf';
import { readFileSync } from 'fs';
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

const __dirname = dirname(fileURLToPath(import.meta.url));
export const pkg = JSON.parse(readFileSync(join(__dirname, '../package.json'), 'utf8'));

const store = new Conf({ projectName: 'uiskill' });

export function getConfig() {
  return {
    apiKey:  process.env.UISKILL_API_KEY || store.get('apiKey'),
    apiUrl:  process.env.UISKILL_API_URL || store.get('apiUrl') || 'https://api.uiskill.dev',
  };
}

export function setConfig(key, value) {
  store.set(key, value);
}

// uiskill config set api-key xxxx
export function configCommand(key, value) {
  const validKeys = ['api-key', 'api-url'];
  if (!validKeys.includes(key)) {
    console.error(`Unknown config key: ${key}. Valid keys: ${validKeys.join(', ')}`);
    process.exit(2);
  }
  setConfig(key.replace('-', 'Key'), value); // api-key → apiKey
  printSuccess(`Set ${key}`);
}
```

---

## Python CLI (Typer)

### Structure

```
your-cli/
├── pyproject.toml
├── src/
│   └── cli/
│       ├── __init__.py
│       ├── main.py         — Typer app + subcommands
│       ├── commands/
│       │   ├── analyze.py
│       │   └── skills.py
│       ├── config.py
│       ├── output.py       — Rich formatting
│       └── api.py
```

### pyproject.toml

```toml
[project]
name = "uiskill"
version = "1.0.0"
requires-python = ">=3.11"
dependencies = [
  "typer>=0.12.0",
  "rich>=13.0.0",
  "httpx>=0.27.0",
  "keyring>=24.0.0",
]

[project.scripts]
uiskill = "src.cli.main:app"
```

### src/cli/main.py

```python
import typer
from .commands.analyze import analyze_app
from .commands.skills import skills_app

app = typer.Typer(
    name="uiskill",
    help="UI intelligence platform CLI",
    no_args_is_help=True,
)

app.add_typer(analyze_app, name="analyze")
app.add_typer(skills_app, name="skills")

if __name__ == "__main__":
    app()
```

### src/cli/commands/analyze.py

```python
import typer
from typing import Optional
from rich.console import Console
from rich.progress import Progress, SpinnerColumn, TextColumn
from ..api import get_client
from ..output import format_skill

analyze_app = typer.Typer()
console = Console()

@analyze_app.command()
def run(
    url: str = typer.Argument(..., help="URL to analyze (include https://)"),
    page_type: str = typer.Option("homepage", "--page-type", "-t", help="homepage|pricing|docs|landing|dashboard"),
    provider: str = typer.Option("claude", "--provider", "-p", help="LLM provider: claude|openai|gemini"),
    output: Optional[str] = typer.Option(None, "--output", "-o", help="Save skill JSON to file"),
    json_output: bool = typer.Option(False, "--json", help="Output as JSON"),
):
    """Analyze a live website and return its UI Skill."""
    from urllib.parse import urlparse
    if not urlparse(url).scheme:
        console.print("[red]✗[/red] Invalid URL. Include https:// e.g. https://stripe.com")
        raise typer.Exit(2)

    client = get_client()

    with Progress(SpinnerColumn(), TextColumn("[progress.description]{task.description}"),
                  console=console, transient=True) as progress:
        progress.add_task(f"Analyzing {url}...", total=None)
        skill = client.analyze(url, page_type=page_type, provider=provider)

    if json_output:
        import json
        print(json.dumps(skill, indent=2))
    else:
        format_skill(skill, console)

    if output:
        import json
        with open(output, 'w') as f:
            json.dump(skill, f, indent=2)
        console.print(f"[green]✓[/green] Saved to {output}")
```

---

## Interactive Prompts

For commands that need user input mid-flow:

### Node.js (prompts library)

```javascript
import prompts from 'prompts';

const response = await prompts([
  {
    type: 'text',
    name: 'apiKey',
    message: 'Enter your UIskill API key:',
    validate: v => v.startsWith('uisk_') ? true : 'Key must start with uisk_'
  },
  {
    type: 'select',
    name: 'provider',
    message: 'Default LLM provider:',
    choices: [
      { title: 'Claude (recommended)', value: 'claude' },
      { title: 'GPT-4o', value: 'openai' },
      { title: 'Gemini Flash (cheapest)', value: 'gemini' },
    ]
  },
]);

if (!response.apiKey) {
  // User cancelled (Ctrl+C)
  process.exit(1);
}
```

### Python (Rich / Typer prompts)

```python
import typer
from rich.prompt import Prompt, Confirm

api_key = Prompt.ask("Enter your API key")
confirmed = Confirm.ask(f"Delete skill for {domain}?", default=False)
if not confirmed:
    raise typer.Abort()
```

---

## CLI Distribution

### npm (Node.js)

```bash
# Local development install
npm link

# Publish
npm publish --access public
npm install -g uiskill   # User install
```

### pip (Python)

```bash
# Development install
pip install -e .

# Publish
python -m build
python -m twine upload dist/*
pip install uiskill      # User install
```

### Homebrew tap (for wide adoption)

```ruby
# Formula: homebrew-tap/Formula/uiskill.rb
class Uiskill < Formula
  desc "UI intelligence platform CLI"
  homepage "https://uiskill.dev"
  url "https://github.com/yourorg/uiskill/archive/v1.0.0.tar.gz"
  sha256 "..."
  license "MIT"
  depends_on "node"

  def install
    system "npm", "install", *Language::Node.std_npm_install_args(libexec)
    bin.install_symlink Dir["#{libexec}/bin/*"]
  end
end
```

---

## CLI Quality Checklist

Before shipping:

**Commands**
- [ ] Every command has a `--help` description
- [ ] Every argument and option has a description string
- [ ] Commands work without arguments (or fail with clear usage error)
- [ ] `--version` works

**Output**
- [ ] Colors disabled when `NO_COLOR=1` or piped
- [ ] `--json` flag returns clean JSON on stdout
- [ ] Error messages go to stderr, not stdout
- [ ] Exit codes are meaningful (not always 0 or 1)

**Config**
- [ ] API keys can be set via env var
- [ ] Config file location documented
- [ ] `doctor` command validates environment

**Distribution**
- [ ] `package.json` has correct `bin` field
- [ ] `#!/usr/bin/env node` at top of bin file
- [ ] README has install and usage examples