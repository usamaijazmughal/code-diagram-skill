# code-diagram

**v1.1.0** | [Changelog](#changelog)

> An AI skill that reads your actual source code and generates Mermaid diagrams.
> Point it at any directory, file, line range, or multiple targets. It detects the language, paradigm, and produces class, sequence, component, and architecture diagrams.

Works with **Claude Code** | **Cursor** | **Gemini CLI** | ChatGPT (experimental)

---

## Quick Start

<table>
<tr><td><b>Claude Code</b></td><td>

```
/plugin marketplace add usamaijazmughal/code-diagram
/plugin install code-diagram
```
Then: `/code-diagram lib/features/payments`

</td></tr>
<tr><td><b>Cursor</b></td><td>

```bash
curl -o .cursorrules https://raw.githubusercontent.com/usamaijazmughal/code-diagram/main/cursor/.cursorrules
```
Then ask: *"Analyze lib/features/payments and generate diagrams"*

</td></tr>
<tr><td><b>Gemini CLI</b></td><td>

```bash
mkdir -p .gemini/commands && curl -o .gemini/commands/code-diagram.md https://raw.githubusercontent.com/usamaijazmughal/code-diagram/main/gemini/.gemini/commands/code-diagram.md
```
Then: `/code-diagram lib/features/payments`

</td></tr>
</table>

Restart your tool after installing. For Claude Code, type `/code-diagram` and it will autocomplete.

---

## Input Modes & Flags

```bash
/code-diagram lib/features/payments                      # directory
/code-diagram lib/auth/bloc.dart                         # single file
/code-diagram @auth_bloc.dart                            # @ reference
/code-diagram lib/auth/bloc.dart:10-50 class             # line range
/code-diagram [lib/auth, lib/payments] sequence           # multiple targets
/code-diagram lib/app sequence rich                      # show endpoints + operations
/code-diagram lib/app sequence high                      # shorthand
/code-diagram lib/app sequence effort=high               # explicit (self-documenting)
/code-diagram lib/app sequence effort=max rich           # exhaustive + endpoints
/code-diagram [lib/auth, lib/payments] effort=high       # 25 reads per target
```

| Input mode | When to use |
|---|---|
| **Directory** | Analyze a full feature or module |
| **Single file** | Focus on one file's internals |
| **Line range** `path:10-50` | Diagram a specific method or class block |
| **Multi-path** `[a, b, c]` | Compare or connect multiple features |
| **Pasted code** | Quick diagram from code in your clipboard |

| Flag | What it does | Default |
|---|---|---|
| `rich` | Show endpoints, DB entities, operations in diagram labels | Off (method-names only, secure) |
| `low` | Quick shallow — 5 reads, 3-hop trace, structure only | — |
| `medium` | Balanced — 15 reads, 8-hop trace | Default |
| `high` | Deep — 25 reads, 15-hop, DI resolution, abstract→concrete, mixin tracking | — |
| `max` | Exhaustive — no read cap, unlimited trace, full DI graph traversal | — |

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
| **2-49 files** | Full analysis, up to 15 reads (medium effort) |
| **50-99 files** | Grep-first, happy-path sequence only |
| **100+ files** | Auto-decompose by subdirectory + integration diagram |
| **Multi-path** | Per-item budget from effort level, cross-path relationship detection |

Use effort levels to control depth. Default is `medium` (cheap, good for most cases):

```bash
/code-diagram lib/features/payments high           # 25 reads, DI resolution + abstract tracing
/code-diagram lib/features/payments max            # no cap, treats as one unit, full DI graph
/code-diagram [lib/auth, lib/payments] max         # exhaustive per target
```

---

## How It Works

```
Discovery  ->  Paradigm  ->  Grep Scan  ->  Scoring  ->  Reading  ->  Diagrams  ->  Insights
 (Glob)       (3 probes)    (2 passes)    (Tier 1/2/3)  (effort)    (dark theme)   (hotspots)
```

1. **Glob** all source files, exclude noise (tests, generated, i18n)
2. **3 grep probes** classify paradigm: OOP / Component / Functional / Mixed
3. **2 batched grep passes** per language extract classes, relationships, HTTP calls, imports, DI
4. **Tier scoring** identifies foundational files using structural signals (not naming)
5. **Selective reading** — only files that grep can't explain (5–25 reads based on effort, unlimited at `max`). Component + arch diagrams need **zero reads**
6. **Mermaid output** with dark theme, 30-node cap, `rect` flow separation
7. **Insights** — hotspots, violations, external deps, budget usage

---

## Cost Benchmarking

The skill is designed to minimize token usage. You can verify this yourself.

### Test matrix

Run these on the same directory, from cheapest to most expensive:

```bash
/code-diagram lib/features/payments component          # zero reads (grep only)
/code-diagram lib/features/payments arch               # zero reads (grep only)
/code-diagram lib/features/payments class              # 3-8 reads
/code-diagram lib/features/payments sequence           # 5-12 reads
/code-diagram lib/features/payments all                # 8-18 reads
/code-diagram lib/features/payments all rich           # same reads, richer labels
/code-diagram lib/features/payments all high           # DI resolution + abstract tracing
/code-diagram lib/features/payments all max rich       # exhaustive + endpoints
```

### Expected read counts

| Diagram type | Expected reads | If higher, investigate |
|---|---|---|
| `component` | **0** | Any reads = wasted |
| `arch` | **0** | Any reads = wasted |
| `class` | 3–8 | Above 10 = over-reading |
| `sequence` | 5–12 | Above 15 = reading sideways |
| `all` (medium) | 8–15 | Above 15 = budget pressure at medium effort |

### Where to find cost data

Every run reports read budget usage in the **Insights** section at the end:
```
Read budget usage: 12 of 15 reads used (medium effort)
```

### Red flags

- `component` or `arch` triggering file reads (should be zero)
- Sequence chain reading sibling files not in the call chain
- Hitting the budget cap (15 at medium, 25 at high) on a feature under 30 files
- `rich` flag increasing read count (it shouldn't — `rich` only changes labels, not reads)
- `medium` effort using more than 15 reads
- `high` effort not resolving abstract→concrete (DI binding should be checked)

---

## Updating

### Claude Code (marketplace)

```
/plugin update code-diagram
```

### Claude Code (manual install)

```bash
curl -o ~/.claude/skills/code-diagram/SKILL.md \
  https://raw.githubusercontent.com/usamaijazmughal/code-diagram/main/skills/code-diagram/SKILL.md
```
Restart Claude Code after updating.

### Cursor

```bash
curl -o .cursorrules \
  https://raw.githubusercontent.com/usamaijazmughal/code-diagram/main/cursor/.cursorrules
```

### Gemini CLI

```bash
curl -o .gemini/commands/code-diagram.md \
  https://raw.githubusercontent.com/usamaijazmughal/code-diagram/main/gemini/.gemini/commands/code-diagram.md
```

### Check version

The version is in the plugin metadata:
- **Claude Code:** Check `plugins/code-diagram/.claude-plugin/plugin.json` or `.claude-plugin/marketplace.json`
- **All tools:** Compare your local file with the [latest on GitHub](https://github.com/usamaijazmughal/code-diagram/blob/main/plugins/code-diagram/.claude-plugin/plugin.json)

Current version: **1.1.0**

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
No. It greps all files (cheap) and only reads the most important files. At default (`medium`): up to 15 reads. At `high`: up to 25. At `max`: unlimited but traces only what's in the call chain. Component and architecture diagrams need zero reads.
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
code-diagram/
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

### v1.1.0
- **Effort levels:** `low`, `medium` (default), `high`, `max` — replaces `deep` flag
- `high`/`max`: DI registry resolution (GetIt, Hilt, Spring, NestJS) to follow abstract→concrete
- `high`/`max`: inheritance search fallback (grep `extends AbstractClass`) when no DI binding found
- `high`/`max`: mixin/composition tracking (Dart `with`, Kotlin `by`, Python multiple inheritance)
- `max`: no read cap, unlimited chain trace, no auto-decompose split, full DI graph traversal
- Unresolvable boundaries explicitly noted in diagrams
- Universal across all languages (Dart, Java, Kotlin, TypeScript, Python, Go, Swift, Rust)
- `deep` kept as alias for `high` (backward compatibility)

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
