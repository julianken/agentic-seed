# INSTANCE.md

<!-- INSTANCE FACTS for this specific product/repo: what it is, its GitHub identity,
its (optional) Figma design file, and its (optional) merge/review infra. AGENTS.md is
the source of truth for PROCESS (how agents work); this file is the source of truth for
INSTANCE (which product, which repo, which Figma file, which merge setup). DESIGN.md
remains the source of truth for design. Keep process rules out of this file — they belong
in AGENTS.md so the process shape stays portable across products.

THIS IS A FILL-IN TEMPLATE. Replace every {{PLACEHOLDER}} with the value declared in
.seed/placeholders.json (project-bootstrap fill-mode does this, or run it by hand). The
two OPTIONAL sections below are gated on optional placeholders: if {{FIGMA_FILE_ID}} is
blank, DELETE the "Design / Figma" section; if {{REVIEW_BOT}} is blank, DELETE the
"Merge / review infra" review-bot bullet (and the whole section if you also have no
Mergify). -->

## What this is
{{PROJECT_NAME}} — {{PROJECT_TAGLINE}}. Built largely by AI coding agents through reviewed, squash-merged PRs.

## Status
**Status: fill per product** (e.g. "in design — pre-code; no `package.json`, build, or CI app checks yet"). This is the lifecycle fact for *this* repo. Build/test/run commands don't exist until they're added to `AGENTS.md` → "Working in the tree"; until then, agents must not claim them (the tool-agnostic rule is `AGENTS.md` → "Agent guardrails" → anti-invention).

## Repo identity
Local folder `{{LOCAL_FOLDER}}/`; GitHub slug `{{OWNER}}/{{REPO}}` — they may differ, so pass the slug to `gh`. Default branch `{{DEFAULT_BRANCH}}`.

## Design / Figma (OPTIONAL — delete this whole section if {{FIGMA_FILE_ID}} is blank)

`DESIGN.md` is the source of truth for design (see AGENTS.md → "Design source of truth" for the authority ranking). The instance facts about *this product's* Figma file live here.

The design system lives in Figma (file `{{FIGMA_FILE_ID}}`). Read it via the Figma MCP **read tools only** — `get_metadata`, `get_design_context`, `get_screenshot`, `get_variable_defs`, `get_code_connect_map`, `get_libraries`, `search_design_system`, `whoami`. **Never** call a write tool (`use_figma`, `create_new_file`, `generate_figma_design`, `generate_diagram`, `upload_assets`, `add_code_connect_map`) against the **canonical design** — the system pages (Foundations, Components, Screens, States, Motion, Annotations) and every canonical design frame are **human-only**: agents read them; a human edits them. **One narrow, documented exception:** the `figma-design` skill (`.claude/skills/figma-design/SKILL.md`, the pre-code `phase-3-figma-design` BUILD stage) may use Figma write tools, but **only** through its scoped WIP/scratch-page workflow — it writes exclusively to a feature's `{{WIP_PAGE_PATTERN}}` work-in-progress page and **never** touches a canonical design frame or system page. Outside that one skill's scoped WIP writes, the write-tool ban above is absolute.

**Authority:** shipped build > `DESIGN.md` > Figma. `DESIGN.md` wins on any design conflict; a live Figma value that disagrees with it does **not** bind the build — it's *drift to reconcile into `DESIGN.md` in a PR*. Never build straight from a live Figma node, and don't paste its raw hexes/Tailwind — translate to `DESIGN.md` tokens. The two do not auto-sync.

**Flow:** for a known node call `get_design_context` directly; for a large/unknown subtree call `get_metadata(<node>)` first to scope, then `get_design_context`; use `get_screenshot` for visual reference. A URL's `?node-id=<n-n>` is the tool's `nodeId: <n:n>` (hyphen → colon).

**Node map** — *per-product node map goes here*: list this product's Figma pages and screens with their node-ids (URL form `https://figma.com/design/{{FIGMA_FILE_ID}}/?node-id=<n-n>`). Node-ids are drift-prone (a frame rename/reorder can renumber them) — the AGENTS.md Update-Triggers row, not the ids, is the safety net. If live Variable reads and Code Connect are unavailable on this Figma plan (`get_variable_defs` → `{}`), treat Figma as visual reference, not a token feed.

## Product concept (canonical noun)

<!-- The SOLE place this product's vocabulary lives. `scripts/check-concept-drift.sh`
and the `.claude/skills/reviewing/` concept-drift pass READ this block and contain NO
product words — so the same portable check works for every instance. FILL IN at bootstrap;
the example values below are deliberately neutral sentinels (REPLACE-AT-BOOTSTRAP) and a
{{PLACEHOLDER}}, NOT a real product term, so the check is inert on the bare template. -->

A product accretes vocabulary as it evolves; when it **refocuses** (its core abstraction is renamed, or a direction is abandoned), stale copy describing the *old* shape can silently recur on the live surfaces a user or the LLM reads as a present-tense product claim. This block pins what the product currently IS so the drift check can catch the rest. See `AGENTS.md` → Update-Triggers (the concept-refocus row) and the `DORMANT:` / `RETAINED:` tag spec.

- **Canonical noun** — the one word for what the product currently IS. One greppable line.
  - `CANONICAL-NOUN: {{PROJECT_NAME}}` *(replace `{{PROJECT_NAME}}` with the live core-abstraction noun at refocus time, e.g. the thing the product produces today)*
- **Retired terms** — words/phrases that describe an *abandoned* shape and must no longer appear as the live product description. One `RETIRED-TERM:` per line, literal-minded (matched case-insensitively, no NLP — a multi-word phrase is fine). The term is read from the text after the marker, up to the first backtick / `<!--` / `*` / `(`, so a line may carry a per-line `concept-drift-ok:` escape (inside an HTML comment) and the retired-terms list won't self-trip the check. This list starts as a neutral sentinel on the bare template; the check treats `REPLACE-AT-BOOTSTRAP` (and any unreplaced `{{PLACEHOLDER}}`) as "not declared yet" and stays inert. Replace it with the product's actual retired terms when a refocus happens (a fresh product has none — leave the sentinel).
  - `RETIRED-TERM: REPLACE-AT-BOOTSTRAP`  <!-- concept-drift-ok: the retired-terms list itself --> *(replace `REPLACE-AT-BOOTSTRAP` with a real retired term, e.g. the abandoned core abstraction; keep the `concept-drift-ok:` escape on the line)*
- This block is the **only** place a product's vocabulary lives. The check script and the reviewer pass read these `CANONICAL-NOUN:` / `RETIRED-TERM:` lines and embed no product words of their own.

The sanctioned escape hatch for a legitimate retired-term mention (history, a deliberately-retained path, a route/code identifier, a roadmap north-star) is a per-line `DORMANT:` / `RETAINED:` / `concept-drift-ok: <reason>` tag — defined in `AGENTS.md` → "DORMANT / RETAINED tag spec".

## Merge / review infra (OPTIONAL — trim to what this repo actually uses)
- **Mergify** (`.mergify.yml`): an approved PR squash-merges through the queue via a standalone `@Mergifyio queue` comment. The merge *method* and its invariants are process — see `.claude/skills/pr-workflow/SKILL.md` and the user-level `mergify-merge-workflow` skill. *(Delete this bullet if the repo does not use Mergify.)*
- **`@{{REVIEW_BOT}}` is the sole non-author reviewer** *(OPTIONAL — delete this bullet if {{REVIEW_BOT}} is blank)*. Direct push to `{{DEFAULT_BRANCH}}` is blocked by a GitHub ruleset requiring 1 fresh approving review per HEAD from a non-author collaborator; the owner (`@{{CODEOWNER}}`, the lone code owner in `.github/CODEOWNERS`) authors PRs and can't self-approve, so `@{{REVIEW_BOT}}` — the only other collaborator — is what unblocks merge.
