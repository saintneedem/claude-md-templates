# CLAUDE.md Principles

> Everything we know about writing effective CLAUDE.md files — from Anthropic's official guidance, HumanLayer's research, Boris Cherny's team, and the open-source community.

---

## The Attention Budget

Claude Code's system prompt already contains **~50 instructions**. Frontier models can reliably follow **~150-200 instructions** total. That means your CLAUDE.md gets roughly 100-150 instruction slots before things start getting ignored.

Worse: Claude Code's harness tells the model your CLAUDE.md *"may or may not be relevant."* If your file has too many instructions, Claude doesn't selectively ignore the bad ones — it starts ignoring **all of them** uniformly.

**The math is simple: every line you add makes every other line less likely to be followed.**

Source: [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## What to Include vs Exclude (Anthropic Official)

This is from Anthropic's own best practices documentation:

| Include | Exclude |
|---------|---------|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred runners | Detailed API documentation (link to it instead) |
| Repository etiquette (branch naming, PRs) | Information that changes frequently |
| Architectural decisions specific to project | Long explanations or tutorials |
| Developer environment quirks | File-by-file descriptions of the codebase |
| Common gotchas or non-obvious behaviors | Self-evident practices like "write clean code" |

**The test:** Before adding a rule, ask: "Could Claude figure this out by reading my code?" If yes, don't add it.

Source: [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices)

---

## Emphasis Keywords Work

Anthropic confirms that adding emphasis to critical rules improves adherence:

- `IMPORTANT:` for rules that must not be skipped
- `YOU MUST` for non-negotiable constraints
- `NEVER` for hard prohibitions (always pair with what to do instead)

Use these sparingly. If every rule is "IMPORTANT," none of them are.

Source: [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices)

---

## The 3-Level Hierarchy

```
~/.claude/CLAUDE.md          -> Global (every project, ~15 lines)
.claude/CLAUDE.md            -> Project (committed to git, 40-60 lines)
.claude/local.md             -> Local (gitignored, ~10 lines)
```

**Global:** Personal preferences that apply everywhere. "Ask before committing." "Run tests after changes." "Keep code simple."

**Project:** Team context. Stack, structure, commands, conventions. This is the high-leverage file — it turns Claude from a generic assistant into a teammate who knows your codebase.

**Local:** Your personal setup. Your terminal, your MCP servers, your editor quirks. Things your teammates don't need.

---

## Module-Specific CLAUDE.md Files (Advanced)

For larger codebases, you can place CLAUDE.md files in subdirectories:

```
.claude/CLAUDE.md              -> Project root (always loaded)
src/auth/CLAUDE.md             -> Auth module (loaded when working in src/auth/)
src/api/CLAUDE.md              -> API module (loaded when working in src/api/)
src/billing/CLAUDE.md          -> Billing module (loaded on demand)
```

Claude Code loads these **on demand** when it's working in that directory. This is the scaling strategy for monorepos and large codebases — keep the root file lean, push domain-specific rules into the modules that need them.

**When to use this:**
- Your root CLAUDE.md is pushing past 80 lines
- Different parts of the codebase have different conventions
- You have complex domain areas (auth, billing, data pipeline) with specific gotchas

**When NOT to use this:**
- Your project is small/medium (one CLAUDE.md is enough)
- Rules apply universally (those go in root)

---

## Progressive Disclosure

Don't stuff everything into CLAUDE.md. Keep task-specific docs in separate files and tell Claude *when* to read them:

```markdown
## References
- For auth flows or StripeErrors: see docs/stripe-guide.md
- For database migrations: see docs/migration-guide.md
- For deployment: see docs/deploy.md
```

This is **dramatically cheaper** than `@docs/stripe-guide.md`, which embeds the entire file into context every single session whether Claude needs it or not.

The mental model: CLAUDE.md is the index. The docs/ folder is the library. Claude pulls books off the shelf when it needs them.

Source: [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [Shrivu Shankar](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)

---

## The Self-Improvement Loop

From Boris Cherny's 10 tips from the Claude Code team (tip #3):

> "Invest in your CLAUDE.md. After every correction, end with: 'Update your CLAUDE.md so you don't make that mistake again.' Claude is eerily good at writing rules for itself. Ruthlessly edit your CLAUDE.md over time. Keep iterating until Claude's mistake rate measurably drops."

This turns CLAUDE.md from a static config into a **living document that gets smarter over time**. One engineer on the team tells Claude to maintain a notes directory for every task, updated after every PR, then points CLAUDE.md at it.

The cycle:
1. Claude makes a mistake
2. You correct it
3. You say: "Update CLAUDE.md so you don't make that mistake again"
4. Claude writes a specific rule
5. You review and edit the rule (make it sharper)
6. The mistake never happens again

Over weeks, your CLAUDE.md accumulates the team's institutional knowledge.

Source: [Boris Cherny's team tips, tip #3](https://x.com/bcherny/status/2017742747067945390) (from the [full thread](https://x.com/bcherny/status/2017742741636321619))

---

## The Verification Loop

Boris's #1 tip for quality: **give Claude a way to verify its own work.**

> "Probably the most important thing to get great results out of Claude Code — give Claude a way to verify its work. If Claude has that feedback loop, it will 2-3x the quality of the final result." — Boris Cherny ([source](https://x.com/bcherny/status/2007179832300581177))

Anthropic's official best practices agree: "Include tests, screenshots, or expected outputs so Claude can check itself. This is the single highest-leverage thing you can do." ([source](https://code.claude.com/docs/en/best-practices))

Put your verification commands in CLAUDE.md so Claude can run them:

```markdown
## Verification
After every change, run in this order:
1. `npx tsc --noEmit` — fix type errors
2. `npm test` — fix failing tests
3. `npm run lint` — fix lint errors
4. `npm run build` — confirm it builds
```

When Claude has a feedback loop, it produces **2-3x better results**. It will run tests, find issues, and fix them before you even review.

Source: [Boris Cherny's personal setup thread](https://x.com/bcherny/status/2007179832300581177), [Anthropic Best Practices](https://code.claude.com/docs/en/best-practices)

---

## Architecture Diagrams (HumanLayer's Pattern)

HumanLayer uses ASCII architecture flow diagrams in their CLAUDE.md:

```
Claude Code -> MCP Protocol -> hlyr -> JSON-RPC -> hld -> Cloud API
```

This is a cheap way to give Claude spatial understanding of your system. One line of ASCII replaces paragraphs of explanation. Especially useful for:
- Monorepos with multiple services
- Request/response flows
- Data pipelines
- Module dependency chains

Source: [HumanLayer's CLAUDE.md](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md)

---

## The "Don't X, Do Y" Rule

Every prohibition should include the alternative:

```markdown
# Bad — Claude doesn't know what to do instead
- Don't use `any` in TypeScript

# Good — Claude knows exactly what to do
- Don't use `any` — use `unknown` and narrow the type
```

Negative-only instructions leave Claude guessing. Always provide the positive alternative.

---

## Never Send an LLM to Do a Linter's Job

These do NOT belong in CLAUDE.md:
- "Use 2-space indentation"
- "Always add trailing commas"
- "Sort imports alphabetically"
- "Use single quotes"

Use a linter/formatter for deterministic rules. Save CLAUDE.md for judgment calls that require understanding context.

Source: [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## Matt Pocock's Plan Loop

A tiny addition that changes planning behavior:

```markdown
## Plan Mode
- Make the plan extremely concise. Sacrifice grammar for the sake of concision.
- At the end of each plan, give me a list of unresolved questions to answer, if any.
```

The "unresolved questions" pattern forces Claude to surface what it doesn't know before proceeding. Two lines, massive impact.

Source: [Matt Pocock — "My AGENTS.md file"](https://www.aihero.dev/my-agents-md-file-for-building-plans-you-actually-read)

---

## Real-World Benchmarks

| Source | Lines | Key Pattern |
|--------|-------|-------------|
| [HumanLayer](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md) | 57 | ASCII diagrams, TODO priority system, progressive disclosure |
| [Boris Cherny's team](https://x.com/bcherny/status/2007179832300581177) | ~83 | Self-improvement, plan mode, verification loop (shared team CLAUDE.md, private repo) |
| [ChrisWiles](https://github.com/ChrisWiles/claude-code-showcase/blob/main/CLAUDE.md) | 80 | Quick Facts block, skill activation mapping |
| [Cloudflare](https://github.com/cloudflare/templates/blob/main/CLAUDE.md) | 230 | Enterprise monorepo — too long for most projects |
| [Anthropic's example](https://code.claude.com/docs/en/best-practices) | ~10 | Deliberately tiny — just code style + workflow |

The sweet spot: **40-80 lines** for a single project. Under 60 is ideal.

---

## Skill Activation Mapping (Advanced)

From ChrisWiles — tell Claude which skill to load for which task:

```markdown
## Skills
- Creating tests -> use `testing-patterns` skill
- Building forms -> use `formik-patterns` skill
- GraphQL operations -> use `graphql-schema` skill
- Debugging issues -> use `systematic-debugging` skill
```

This is lightweight progressive disclosure that costs almost nothing in context.

Source: [ChrisWiles/claude-code-showcase](https://github.com/ChrisWiles/claude-code-showcase)

---

## TODO Priority System (HumanLayer's Pattern)

Standardize how TODOs are annotated across the team:

```markdown
## TODO Annotations
- `TODO(0)`: Critical — never merge with these
- `TODO(1)`: High — architectural flaws, major bugs
- `TODO(2)`: Medium — minor bugs, missing features
- `TODO(3)`: Low — polish, tests, documentation
- `TODO(4)`: Questions/investigations needed
- `PERF`: Performance optimization opportunities
```

Claude will use these consistently when writing code if they're in your CLAUDE.md.

Source: [HumanLayer's CLAUDE.md](https://github.com/humanlayer/humanlayer/blob/main/CLAUDE.md)

---

## Anti-Patterns

| Anti-pattern | Why it fails | What to do instead |
|---|---|---|
| Relying on `/init` without editing | The generated file is generic and often too long ([HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)) | Use `/init` as a starting point ([Anthropic recommends it](https://code.claude.com/docs/en/best-practices)), then ruthlessly edit down |
| Over 100 lines | Instructions get ignored uniformly | Keep root under 60, use module-specific files |
| Personality instructions | "Be a senior engineer" wastes tokens | Claude already has strong system-level behavior |
| @-mentioning docs | Embeds entire file every session | Pitch Claude on *when* to read: "For X, see docs/Y.md" |
| Code snippets | They go stale fast | Use file:line references instead |
| Formatting rules | Linters are deterministic, LLMs aren't | Use prettier/eslint/ruff |
| Negative-only rules | Claude doesn't know what to do | "Don't X — do Y instead" |
| Duplicate rules | Wastes attention budget | Each rule in exactly one place |
| Treating it as documentation | It's an onboarding brief, not a wiki | Keep it lean, link to docs/ |

---

## Further Reading

- [Anthropic — "Effective Context Engineering for AI Agents"](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Context rot, attention budget, just-in-time context
- [Anthropic — "Claude Code Best Practices"](https://code.claude.com/docs/en/best-practices) — Official guidance
- [HumanLayer — "Writing a Good CLAUDE.md"](https://www.humanlayer.dev/blog/writing-a-good-claude-md) — Instruction limits, progressive disclosure, leverage diagram
- [Boris Cherny's team tips](https://x.com/bcherny/status/2017742741636321619) — 10 tips from the Claude Code team (Jan 31, 2026)
- [Boris Cherny's personal setup](https://x.com/bcherny/status/2007179832300581177) — How the creator uses Claude Code (Jan 2, 2026)
- [Boris Cherny's setup (summary)](https://paddo.dev/blog/how-boris-uses-claude-code/) — paddo.dev's analysis of Boris's workflow
- [Shrivu Shankar — "How I Use Every Claude Code Feature"](https://blog.sshh.io/p/how-i-use-every-claude-code-feature) — Practitioner deep dive
- [josix/awesome-claude-md](https://github.com/josix/awesome-claude-md) — Curated collection of real CLAUDE.md files from open-source projects
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — The main awesome list for Claude Code resources
- [Matt Pocock — "My AGENTS.md file"](https://www.aihero.dev/my-agents-md-file-for-building-plans-you-actually-read) — Plan loop rules

---

*From the CLAUDE.md Starter Kit by [Claude Code Camp](https://claudecodecamp.com)*
