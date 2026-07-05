## MANDATORY: Never Commit or Push Without Explicit Instruction

**NEVER run `git commit` or `git push`.** Make file edits only and stop. Tell the user what to commit and let them do it themselves. The only exception is when the user explicitly says "commit" (or "push") in that request — a general task to change code is NOT permission to commit. No exceptions. No rationalizing.

## Core principles (Karpathy-inspired)

- **Think before coding**: state assumptions explicitly; if genuinely ambiguous, present the interpretations instead of silently picking one. Push back if a simpler approach exists.
- **Simplicity first**: minimum code that solves the problem. No speculative flexibility, no unrequested abstractions.
- **Surgical changes**: touch only what the task requires. Don't refactor, reformat, or "improve" adjacent code. Remove only imports/vars your own change orphaned. Match existing style.
- **Goal-driven execution**: turn tasks into verifiable success criteria (e.g. "write a failing test, then make it pass"). State a short plan with a verify step per item for non-trivial tasks.

## MANDATORY: No Explore Agents When Tokensave Is Available

- Other structural tools (language servers, semantic-navigation MCPs) are valid, sometimes-better alternatives — use them when their strengths fit; prefer tokensave for small, targeted lookups.
- **Explore agent** → last resort for code research. Don't cold-start `Agent(subagent_type=Explore)` for "where is X / what calls this" — a structural tool (TokenSave, Serena, rust-analyzer) answers cheaper. Fall back to Explore only when no structural tool fits (not a code question, or the index is unavailable); if you do, tell it to use the project's structural tools instead of raw Read/grep.


## Tool routing (use the right layer)

- **TokenSave** → primary and default retrieval/navigation layer, especially for small, short tasks: analysis, searching, and locating small fixes. Reach for it first. Use its vector/semantic search rather than recursive scans or broad greps, and prefer its symbol-level operations over whole-file reads (`tokensave_context`, `tokensave_search`, `tokensave_callers`, `tokensave_impact`). Note: its index goes stale across branch switches. For structural queries the tools don't expose, query `.tokensave/tokensave.db` directly (tables: `nodes`, `edges`, `files`).
- **Serena** → complementary code navigation (references, call hierarchies) where TokenSave doesn't cover it. Prefer it for larger / multi-step tasks that span many files and plan-mode, since it carries project context across the task.
- **rust-analyzer** → Rust types, signatures, diagnostics, navigation. Don't guess types or signatures.
- **context7** (if available) → pull up-to-date, version-specific docs and API signatures for external libraries/crates before using them. Prevents outdated or hallucinated APIs; cheaper than reading vendored source.
- **Svelte MCP** → for any Svelte 5 / SvelteKit code:
  - Always run `svelte-autofixer` on generated Svelte code *before* returning it.
  - Try own knowledge + autofixer first; only call the docs tool (`list-sections` → `get-documentation`) when needed, since it is token-intensive.
- **Playwright** → E2E/browser tests only when the task involves them. Prefer targeted specs over full runs.
- **ast-grep (`sg`)** → structural (AST-level) search & replace for Rust/TS refactors; more precise and lower-noise than text grep.
- **RTK** → optional, only when actually executing shell commands/scripts. Wrap noisy output (`rtk cargo test`, `rtk git status`) to compress it. Not a replacement for TokenSave — it only touches Bash output, not code retrieval. Bypass with `rtk proxy <cmd>` when full output is needed (e.g. debugging Playwright/E2E failures).
- **CLI availability**: `uvx` (run Python tools ephemerally) and `node`/`npx` are available — prefer ephemeral invocation over global installs.

## Rust

- Run `cargo check` (not `cargo build`) for fast feedback; use rust-analyzer diagnostics before proposing fixes.
- Run `clippy` only when requested, or once before finishing a larger change.
- Preserve `rustfmt` formatting; don't hand-reformat.
- Respect existing error-handling patterns (`thiserror`/`anyhow` as already used in the crate) rather than introducing new ones.

## Frontend / Svelte

- Match the existing setup (Svelte 5 runes, Vite; not SvelteKit unless the project already uses it).
- Component quality gate = `svelte-autofixer` before handing code back.
- Keep components small; don't add state-management libraries that aren't already present.

## Token efficiency

- Retrieve via TokenSave; read the minimum necessary; prefer symbol-level reads/edits over whole files. Don't reread files already inspected this session. Search before reading.
- Batch related edits instead of many small round-trips.
- When running shell commands, let RTK compress the output; don't paste raw multi-hundred-line dumps.
- Keep responses concise; don't repeat the prompt or explain unchanged code. Stop once the task is verifiably complete.

## Planning

- Short plan (steps + verify criteria) only for non-trivial tasks.
- Ask only if genuinely blocked; otherwise proceed autonomously and flag assumptions inline.

## Testing

- Run only tests relevant to the modified module. Don't run the full suite unless requested or the change is cross-cutting.

## Safety

- Never delete files or drop pre-existing dead code without confirmation — mention it, don't remove it unasked.
- Preserve public APIs unless a breaking change was explicitly requested.