# code-diagram

**v1.0.2** | [Changelog](#changelog)

> An AI skill that reads your actual source code and generates Mermaid diagrams.
> Point it at any directory, file, line range, or multiple targets. It detects the language, paradigm, and produces class, sequence, component, and architecture diagrams.

Works with **Claude Code** | **Cursor** | **Gemini CLI** | ChatGPT (experimental)

---

## Quick Start

<table>
<tr><td><b>Claude Code</b></td><td>

```
/plugin marketplace add usamaijazmughal/code-diagram-skill
/plugin install code-diagram
```
Then: `/code-diagram lib/features/payments`

</td></tr>
<tr><td><b>Cursor</b></td><td>

```bash
curl -o .cursorrules https://raw.githubusercontent.com/usamaijazmughal/code-diagram-skill/main/cursor/.cursorrules
```
Then ask: *"Analyze lib/features/payments and generate diagrams"*

</td></tr>
<tr><td><b>Gemini CLI</b></td><td>

```bash
mkdir -p .gemini/commands && curl -o .gemini/commands/code-diagram.md https://raw.githubusercontent.com/usamaijazmughal/code-diagram-skill/main/gemini/.gemini/commands/code-diagram.md
```
Then: `/code-diagram lib/features/payments`

</td></tr>
</table>

Restart your tool after installing. For Claude Code, type `/code-diagram` and it will autocomplete.

---

## Input Modes

```bash
/code-diagram lib/features/payments                      # directory
/code-diagram lib/auth/bloc.dart                         # single file
/code-diagram @auth_bloc.dart                            # @ reference
/code-diagram lib/auth/bloc.dart:10-50 class             # line range
/code-diagram [lib/auth, lib/payments] sequence           # multiple targets
/code-diagram [lib/auth, lib/pay/bloc.dart, @file.dart]  # mixed types
```

| Mode | When to use |
|---|---|
| **Directory** | Analyze a full feature or module |
| **Single file** | Focus on one file's internals |
| **Line range** `path:10-50` | Diagram a specific method or class block |
| **Multi-path** `[a, b, c]` | Compare or connect multiple features |
| **Pasted code** | Quick diagram from code in your clipboard |

---

## Diagram Types

| Type | Command | What it shows |
|---|---|---|
| **Class** | `class` | Classes, interfaces, inheritance, composition, dependency |
| **Sequence** | `sequence` | Execution flow: entry point through layers to external API |
| **Component** | `component` | External dependencies: HTTP endpoints, packages, services |
| **Architecture** | `arch` | Layer boundaries and cross-layer relationships |
| **All** | `all` (default) | All four diagrams |

Each diagram adapts to the detected paradigm:

| Paradigm | `class` becomes | `sequence` becomes |
|---|---|---|
| OOP | Class diagram | Method call chain |
| Component (React/Vue) | Component tree | Data flow |
| Functional (Go/Express) | Module map | Call graph |

---

## Flow Modes

| Mode | Behavior | Best for |
|---|---|---|
| `full` (default) | One unified diagram per type | Documentation, onboarding |
| `split` | Multiple focused diagrams | Large features, presentations |

```bash
/code-diagram lib/app class split    # one diagram per layer
/code-diagram lib/app sequence split # one diagram per flow
```

---

## Supported Languages

| Language | Status | Paradigms |
|---|---|---|
| Dart / Flutter | Tested | OOP |
| TypeScript / JavaScript | Tested | OOP, Component, Functional |
| Java / Kotlin | Tested | OOP |
| Python | Covered | OOP, Functional |
| Go | Covered | Functional, Structural |
| Swift | Covered | OOP |
| Rust | Covered | Structural, Trait-based |
| Vue / Angular / Svelte | Covered | Component, Mixed |

**Tested** = validated on production codebases. **Covered** = patterns implemented, community validation welcome.

---

## Scale Handling

| Size | Strategy |
|---|---|
| **1 file** | Read directly, focused diagram |
| **2-49 files** | Full analysis, 20-file read budget |
| **50-99 files** | Grep-first, happy-path sequence only |
| **100+ files** | Auto-decompose by subdirectory + integration diagram |
| **Multi-path** | Per-item budget, cross-path relationship detection |

When the budget is scaled down (100+ files or multi-path), the skill asks you to choose:

```
Budget options:
  1. Balanced (recommended) — proportional reads, cheaper
  2. Deep — 20 reads per target, full depth, more expensive
Which mode? (1/2, default: 1)
```

---

## How It Works

```
Discovery  ->  Paradigm  ->  Grep Scan  ->  Scoring  ->  Reading  ->  Diagrams  ->  Insights
 (Glob)       (3 probes)    (2 passes)    (Tier 1/2/3)  (max 20)    (dark theme)   (hotspots)
```

1. **Glob** all source files, exclude noise (tests, generated, i18n)
2. **3 grep probes** classify paradigm: OOP / Component / Functional / Mixed
3. **2 batched grep passes** per language extract classes, relationships, HTTP calls, imports, DI
4. **Tier scoring** identifies foundational files using structural signals (not naming)
5. **Selective reading** — only files that grep can't explain (max 20). Component + arch diagrams need **zero reads**
6. **Mermaid output** with dark theme, 30-node cap, `rect` flow separation
7. **Insights** — hotspots, violations, external deps, budget usage

---

## Updating

### Claude Code (marketplace)

```
/plugin update code-diagram
```

### Claude Code (manual install)

```bash
curl -o ~/.claude/skills/code-diagram/SKILL.md \
  https://raw.githubusercontent.com/usamaijazmughal/code-diagram-skill/main/skills/code-diagram/SKILL.md
```
Restart Claude Code after updating.

### Cursor

```bash
curl -o .cursorrules \
  https://raw.githubusercontent.com/usamaijazmughal/code-diagram-skill/main/cursor/.cursorrules
```

### Gemini CLI

```bash
curl -o .gemini/commands/code-diagram.md \
  https://raw.githubusercontent.com/usamaijazmughal/code-diagram-skill/main/gemini/.gemini/commands/code-diagram.md
```

### Check version

The version is in the plugin metadata:
- **Claude Code:** Check `plugins/code-diagram/.claude-plugin/plugin.json` or `.claude-plugin/marketplace.json`
- **All tools:** Compare your local file with the [latest on GitHub](https://github.com/usamaijazmughal/code-diagram-skill/blob/main/plugins/code-diagram/.claude-plugin/plugin.json)

Current version: **1.0.2**

---

## Where to Render Diagrams

| Renderer | How |
|---|---|
| [mermaid.live](https://mermaid.live) | Paste and render instantly |
| **GitHub** | Paste in any `.md` file, PR, or issue |
| **GitLab** | Native Mermaid in markdown |
| **Notion** | Code block with `mermaid` language |
| **VS Code** | Install Mermaid preview extension |

---

## FAQ

<details>
<summary><b>Does it read every file?</b></summary>
No. It greps all files (cheap) and only reads the most important 10-20. Component and architecture diagrams need zero reads.
</details>

<details>
<summary><b>What if my base classes aren't named "Base"?</b></summary>
The skill uses structural signals, not naming conventions. A class named <code>Payments</code> that 12 files extend outranks <code>BaseHelper</code> that nothing extends.
</details>

<details>
<summary><b>Can I use it on a monorepo?</b></summary>
Yes, but point at a specific feature directory. Or use multi-path: <code>[packages/auth, packages/billing]</code>
</details>

<details>
<summary><b>What Mermaid version?</b></summary>
Standard syntax. <code>rect</code> blocks need Mermaid 9.3+. Dark theme needs 9+. Avoids <code>namespace</code> (10.3+ only).
</details>

<details>
<summary><b>Wrong diagram output?</b></summary>
Open an issue with: language, paradigm, feature size, and what was wrong.
</details>

---

## Repository Structure

```
code-diagram-skill/
  skills/code-diagram/SKILL.md             # Claude Code (manual install)
  plugins/code-diagram/                     # Claude Code (marketplace install)
    .claude-plugin/plugin.json
    skills/code-diagram/SKILL.md
  cursor/.cursorrules                       # Cursor
  gemini/.gemini/commands/code-diagram.md   # Gemini CLI
  chatgpt/SYSTEM_PROMPT.md                  # ChatGPT (experimental)
  .claude-plugin/marketplace.json           # Marketplace registry
```

---

## Changelog

### v1.0.2
- Detail level prompt: method names (recommended, secure) vs rich details (opt-in)
- 9 external operation categories: DB, cache, storage, queue, WebSocket, push, auth, analytics, gRPC
- Rich mode: full endpoint paths, `READ/WRITE entity` for DB, compact operations for all others
- Applies to ALL diagram types: sequence, class, component, architecture
- DB security: never shows query text, columns, or schema in any mode
- Reference table for sequence diagrams with 15+ steps (rich mode only)
- 3 batched composite greps (not 9 separate) for cost-effectiveness

### v1.0.1
- Added 5 input modes: line range, multi-path, pasted code, file validation, @ reference docs
- Multi-path supports mixed types: `[directory, file, @ref, path:10-50]`
- Balanced/deep budget prompt for auto-decompose and multi-path
- Cross-path relationship detection in multi-path mode
- Phase applicability matrix for all input modes
- 8 critical/high audit fixes (budget conflicts, regex, parsing, platform sync)
- Synced all platforms: Cursor, Gemini CLI, ChatGPT

### v1.0.0
- Initial release: 5-phase pipeline, 4 diagram types, 8 languages, 3 paradigms
- Dark theme, grep-first strategy, structural scoring, auto-decompose
- Ported to Claude Code, Cursor, Gemini CLI, ChatGPT

---

## Contributing

1. **Test it** on your codebase (especially Python, Go, Swift, Rust)
2. **Report issues** with language, paradigm, and feature size
3. **PRs welcome** for new languages, paradigm patterns, or tool ports

---

## License

MIT

---

Built by [Muhammad Usama Ijaz](https://github.com/usamaijazmughal)
