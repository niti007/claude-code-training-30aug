# Claude Code — Advanced Session (30th Aug)

Format for each use case: **Problem statement** → for every prompt: **short plain-English "why" first**, then **the exact prompt to paste**.

---

## Use Case 1 — Bootstrapping Claude Code in a brand-new project (TDD + minimalism walkthrough)

**Problem statement:** You're starting a brand new .NET project from scratch (a small "OrderPricing" library). Show the team how to onboard Claude *before* writing code, drive the coding with TDD so every change is provably correct, and install discipline that stops Claude from over-engineering or over-writing code.

**Prompts to run, in order:**

1. Terminal setup (not a Claude prompt):
   `mkdir OrderPricing && cd OrderPricing && git init`

2. **Why:** Before any code exists, Claude should write its own onboarding notes — build/test commands, directory map, conventions — so nobody (human or Claude) has to re-explain basics next time.
   > "This is a brand new .NET class library called OrderPricing. Before writing any code, set up a CLAUDE.md with build/test commands (we'll use dotnet test), a directory map, and coding conventions for this repo. Also set up .claude/settings.json with a sensible allowlist for dotnet and git commands."

3. **Why:** Install a minimalism discipline up front, so every later prompt in this session is already constrained to not over-build. This is where **Ponytail** comes in — a plugin that forces Claude through a "does this really need to exist?" checklist before writing anything, prioritizing reuse and minimum code over invented abstractions.
   > "/plugin marketplace add DietrichGebert/ponytail" then "/plugin install ponytail@ponytail" then "/ponytail full"
   Explain to the room: Ponytail makes Claude check, in order — does this need to exist at all (YAGNI)? Is it already in the codebase? Is it in the standard library? A native platform feature? An already-installed dependency? Can it be one line? Only then does it write new code. It never cuts corners on security, validation, or accessibility — only on unnecessary bulk. Its own benchmarks report ~54% less code and ~20% lower cost on comparable tasks.

4. **Why:** We write the "accusation" first — a test describing behavior that doesn't exist yet — so there's a concrete, checkable definition of "done" before any implementation exists.
   > "We're going to build this test-first. First, write a failing test in OrderPricing.Tests for a `PriceCalculator.CalculateTotal` method that applies a 10% discount for orders over $100. Don't write the implementation yet."

5. Run `dotnet test` — show it fails.

6. **Why:** Now, and only now, Claude writes the minimum code needed to satisfy the test — Ponytail's discipline plus TDD's discipline reinforce each other here: no code without a red test demanding it, and no more code than the test demands.
   > "Now implement just enough in PriceCalculator to make that test pass. Don't add anything beyond what the test requires."

7. Run `dotnet test` — show it passes.

8. **Why:** Repeat the loop with a second rule, showing this isn't a one-off trick — it's how every feature in this project should get built.
   > "Add a second failing test for orders under $100 getting no discount, then implement it."

9. **Why:** TDD doesn't have to stop at unit tests — extend the same red/green discipline to the UI layer using the `/webapp-testing` skill (Playwright integration). It connects Claude Code to a real headless browser: Claude can write a UI test suite for a local frontend, run it, watch it fail, fix its own code or test, and iterate — with none of the usual Playwright boilerplate setup.
   > "Using the webapp-testing skill, write a Playwright test that opens our local OrderPricing demo page, enters an order total of $150, and asserts the displayed discount is 10%. Run it, and if it fails, fix whichever side (app or test) is wrong."

10. **Why:** At the end of a work session, distill only the non-obvious lessons learned — corrections, gotchas, conventions — into a short note, instead of re-explaining them next session. This mirrors the "continuous learning" pattern from `everything-claude-code`'s skill set and its `/checkpoint`/`/learn` commands.
    > "Create a skill called `reflect`. When invoked, it should scan this conversation for any non-obvious decisions or corrections I gave you (e.g. conventions, gotchas, things you got wrong and I fixed), and write a short, compressed summary into CLAUDE.md or a memory file — not a raw transcript dump. It should be something I run manually at the end of a work session, not automatically every turn."

11. **Why:** Run it once, at the natural end of the session, so we pay the token cost only when there's something worth keeping.
    > "/reflect"

12. **Why:** Everything so far was done manually, step by step, so the team sees each moving part. **Superpowers** packages this entire discipline — brainstorm, plan in small verifiable tasks, dispatch subagents per task with two-stage review, enforce TDD red-green-refactor, review, then merge — into one installed methodology, so a real feature doesn't need you to remember to ask for each stage.
    > "/plugin marketplace add obra/superpowers-marketplace" then "/plugin install superpowers@superpowers-marketplace"
    > "Using Superpowers, plan and implement a new feature: OrderPricing should support a stacked promo-code discount on top of the existing bulk discount. Let the framework brainstorm, plan, and execute it."

13. **Why:** From `everything-claude-code`, borrow two more specifically useful agents for this stage: `build-error-resolver` for when Claude's own generated code breaks the build, and `tdd-guide` to keep enforcing red-green-refactor discipline even inside Superpowers' automated flow. Also show `/checkpoint`, which preserves verification state so a session can be paused and resumed without replaying all this context.
    > "Add the build-error-resolver and tdd-guide subagents from everything-claude-code to this project, and run /checkpoint now so I can safely pause and resume this session later without re-explaining everything."

14. **Why:** Instead of trusting Claude to remember "always write the test first," a **PreToolUse hook** can enforce it mechanically — block any Edit/Write that adds implementation code to PriceCalculator if there's no corresponding failing test yet. This is the same "deterministic beats remembered" principle used later for the cross-repo hook: a rule worth enforcing is worth enforcing automatically, not just written down and hoped for.
    > "Add a PreToolUse hook that blocks edits to PriceCalculator.cs unless a corresponding test file was modified more recently in this session — enforcing test-first mechanically instead of relying on me remembering to ask for it."

15. **Why, token-optimization angle:** A skill like `webapp-testing` or the `reflect` skill shouldn't be loaded into context on every single turn "just in case" — skills are designed to load only when their trigger conditions are actually relevant, the same lazy-loading principle Use Case 2 applies to CLAUDE.md files. Point out explicitly: having 10 skills installed costs nothing on a turn that doesn't need any of them, unlike a monolithic CLAUDE.md that's read in full every time.
    > "List which skills are currently installed in this project and confirm none of them are being loaded into context on turns where they're not relevant — show me how skill loading differs from CLAUDE.md loading in terms of when each one actually costs tokens."

16. **Why:** A human changelog and a test suite serve the same purpose from different angles — the tests prove *that* behavior is correct, the changelog explains *why* it changed, for the next person (or the next Claude session) who wasn't in this conversation. Have Claude maintain CHANGELOG.md as part of the same TDD loop, not as an afterthought — one entry per feature slice, written right after its tests go green.
    > "After each feature slice that passes its tests (like the promo-code discount just now), add a dated entry to CHANGELOG.md summarizing what changed and why — one or two lines, written for a teammate who wasn't in this session."

17. **Why:** Just like test-first was worth enforcing mechanically (step 14), a changelog entry is easy to forget under deadline pressure — pair it with a hook so it happens automatically instead of depending on Claude remembering to ask.
    > "Add a PostToolUse hook that reminds me (or auto-drafts an entry) to update CHANGELOG.md whenever a feature branch's tests go from failing to passing, so it can't quietly be skipped."

**What to point out:**
- CLAUDE.md is written before any code — short, not a dump.
- Ponytail installs a standing "don't over-build" discipline for the rest of the session.
- TDD (unit-level, via dotnet test) and TDD-for-UI (via `/webapp-testing` + Playwright) are the same discipline applied at two layers: no code accepted without a failing test proving it was needed.
- `reflect` is run manually at natural checkpoints, not every turn — that's the token-saving discipline, echoing `everything-claude-code`'s `/checkpoint` and `/learn` pattern.
- Superpowers is what all these individual disciplines (TDD, minimalism, checkpointing) look like once packaged into one enforced methodology instead of you asking for each piece by hand every time.
- The PreToolUse hook turns "please always write the test first" from a hoped-for instruction into a mechanically enforced rule — hooks are for the things you truly cannot afford Claude to forget.
- Skills only cost tokens when actually loaded/relevant — that's a different (and cheaper) cost model than an always-loaded CLAUDE.md, and it's why deep, rarely-needed knowledge belongs in a skill, not the root file.
- CHANGELOG.md is the human-readable "why," CLAUDE.md/`reflect` memory is the machine-readable "what to remember" — both get written at the same natural checkpoint (tests green), and both are cheap to skip under pressure, which is exactly why the hook exists.

---

## Use Case 2 — Claude Code on a massive existing codebase

**Problem statement:** You inherit a huge, real-world .NET codebase. Running `/init` naively either produces a giant, low-signal file, or one that glosses over entire subsystems. Show how to fix that, and how to give Claude durable memory of what it learns about this codebase across sessions.

**Prompts to run, in order:**

1. Terminal setup (not a Claude prompt):
   `git clone --depth 1 https://github.com/dotnet/runtime && cd runtime`

2. **Why:** See the naive, default outcome first, so the fix that follows actually lands.
   > "/init"
   Show the resulting CLAUDE.md live — point out it's either huge and unreadable, generic boilerplate, or silently skips major subsystems (CoreCLR, mono, libraries, installer are near-independent projects inside this one repo).

3. **Why:** A single flat CLAUDE.md forces Claude to load the same huge context on every task regardless of relevance. A slim root CLAUDE.md acts as an index; directory-scoped CLAUDE.md files load only when Claude is actually working inside that subsystem — cost becomes proportional to where the work is happening.
   > "This repo is too large for one CLAUDE.md. Instead, create a slim root CLAUDE.md that's just an index/map of subsystems (src/coreclr, src/mono, src/libraries, src/installer), and propose directory-scoped CLAUDE.md files for 2 of those subsystems so their detail only loads when I'm working there."

4. **Why:** Generated code, build output, and vendored/third-party folders add scanning cost with zero signal. Excluding them means Claude's attention goes only to code people actually wrote and maintain.
   > "Also propose a .claudeignore-style exclusion list for generated/build/vendored directories in this repo so exploration isn't wasted scanning them."

5. **Why:** Deep domain knowledge about one subsystem doesn't belong in an always-loaded file — it belongs in a skill that loads on demand, the same "specialist called in only when needed" pattern from `everything-claude-code`'s skill library (backend patterns, frontend patterns are organized this way there too).
   > "For src/libraries specifically, instead of dumping API details into CLAUDE.md, create a skill that Claude loads on demand when working on library code — show me the skill file."

6. **Why:** CLAUDE.md files reset your understanding to the last time someone wrote them down. A codebase this size also benefits from a running memory that accumulates what Claude has *learned* while actually working in it across many sessions — not just static docs. This is what **Supermemory** (a local-runnable memory/context engine with a Memory API) is for: durable, queryable memory that persists insights ("this subsystem's tests are flaky for reason X", "this API is deprecated in favor of Y") across sessions without re-deriving them each time.
   > "Set up Supermemory locally as a memory store for this project. After I work in src/libraries today, write a summary of what we learned about its test flakiness and API surface into Supermemory, tagged by subsystem, so a future session can query it instead of re-exploring the code."

7. **Why:** Real repos also carry unwritten "tribal knowledge" — conventions repeated everywhere with no comment explaining why. Surfacing these prevents Claude from confidently doing the wrong thing because nothing on paper told it not to.
   > "Are there any conventions in src/libraries that look like unwritten team rules — patterns repeated everywhere with no comment explaining why? List them so we can decide whether to document them."

8. **Why:** A word of caution from `everything-claude-code`'s own guidance: don't enable every MCP server simultaneously — a 200k context window can shrink to as little as 70k with too many tools enabled. Their own rule of thumb: keep a larger library configured (20-30 MCPs) but fewer than 10 *active* per project. The same "only load what's relevant" principle from the CLAUDE.md/skills split applies to MCP servers and tool access too.
   > "List the MCP servers currently configured for this project and flag any that aren't needed for the subsystem I'm actively working in."

9. **Why:** Long working sessions on a huge repo accumulate stale context (finished subtasks, dead ends, superseded plans) that just sits there costing tokens on every turn. Rather than a full new session (losing everything) or doing nothing (paying for junk context forever), `strategic-compact` gives a manual, on-demand compaction — you decide when the moment is right to trim.
   > "Run strategic-compact now to trim this session's context — we're done exploring src/libraries and about to move to a different subsystem."

10. **Why:** Pair Supermemory (step 6) with `everything-claude-code`'s `memory-persistence` hooks so this isn't a manual "remember to save" step — the hook fires automatically at natural session boundaries, saving/loading context so nothing learned today has to be re-explained tomorrow.
    > "Add the memory-persistence hook pattern from everything-claude-code, wired to write into Supermemory automatically at session end, so tomorrow's session loads today's learnings without me asking for it."

11. **Why:** Just like the codebase evolves, the CLAUDE.md structure needs a rule for when to split or merge scoped files further, otherwise today's fix decays back into one stale file within a few months.
    > "Suggest a lightweight rule for when a subsystem's CLAUDE.md should be split further or merged back — e.g. based on directory size or how often it comes up in tasks."

12. **Why:** A huge codebase like this depends on external NuGet packages and APIs Claude wasn't trained on or half-remembers incorrectly. Instead of Claude guessing from stale training knowledge (or dumping an entire doc site into context to be sure), **ref-tools-mcp** is an MCP server built specifically to fetch just the relevant reference snippet for a library/API on demand — the same "load only what's needed" principle applied to third-party documentation instead of your own codebase.
    > "Add the ref-tools MCP server to this project. Next time we need to check the exact signature or behavior of a System.* or third-party API we're not sure about, use it to pull just the relevant reference instead of guessing or fetching a full doc page."

13. **Why:** Sometimes you genuinely want to hand a chunk of code to an LLM as one big blob — e.g. asking an *external* model to do a one-off architectural review of a subsystem. **Repomix** packs a repo (or a chosen subset) into a single AI-friendly file for exactly that. The lesson here is contrast: used on the *whole* dotnet/runtime repo it recreates the exact "unreadable binder" problem from step 2 — used *scoped to one subsystem*, it's a legitimate, deliberate tool for a specific one-off task, not a substitute for CLAUDE.md/skills as Claude's everyday working context.
    > "Use Repomix to pack only the src/libraries/System.Text.Json subsystem into a single file, so we can hand it to an external reviewer/LLM for a one-off deep-dive — not the whole repo."

14. **Why:** Rather than one general-purpose session accumulating context from every subsystem it's ever touched, dispatch `everything-claude-code`'s `architect` and `refactor-cleaner` subagents *only* for the specific subsystem under review right now — each spins up with a clean, narrow context, does its scoped job, and hands back a short report, instead of the main session's context growing every time a different subsystem gets looked at.
    > "Use the architect subagent from everything-claude-code to review src/libraries/System.Text.Json's structure in isolation, and refactor-cleaner to flag dead code in it — both scoped to just this subsystem, reporting back a summary rather than dumping their full working context into my main session."

15. **Why:** A SessionStart hook can automate the "only load the relevant scoped CLAUDE.md" decision instead of you manually telling Claude which subsystem you're in each time — it inspects which directory you're working in (e.g. the last-edited file's path) and loads only that subsystem's CLAUDE.md/skill, keeping the always-on cost to just the slim root index.
    > "Add a SessionStart hook that detects which subsystem directory I'm actively working in based on recently modified files, and only surfaces that subsystem's scoped CLAUDE.md — not all of them — at the start of each session."

**As the codebase keeps growing — what changes next (talk through, don't necessarily demo all live):**

16. **Why:** Two scoped CLAUDE.md files works fine; twenty stops working the same way, because now the *root* index itself risks becoming the new bloated binder. The fix is the same idea applied one level up — the index needs its own discipline for staying an index and not creeping into detail.
    > "Audit our root CLAUDE.md as if 15 more subsystems had scoped files added since today. Rewrite it as a rule: what belongs in the root index vs. what must live one level down, so the root file has a hard reason to stay small no matter how many subsystems we add."

17. **Why:** At real scale, nobody remembers to keep every scoped CLAUDE.md in sync with the code it describes — they go stale silently, and a stale CLAUDE.md is worse than no CLAUDE.md because it actively misleads. A hook can flag drift instead of hoping someone notices in review.
    > "Add a hook that flags when a subsystem's code has changed significantly (e.g. many files touched) since its scoped CLAUDE.md was last updated, so staleness gets caught instead of silently accumulating."

18. **Why:** Some organizations don't just have one huge repo — they have dozens of repos that all share the same conventions (coding style, review process, security rules). Rewriting the same CLAUDE.md content in every repo doesn't scale either. The fix is a shared, versioned base (a plugin or a small internal package of skills/rules) that every repo's CLAUDE.md references instead of restating.
    > "Suppose we have 30 repos like this one across the org, all sharing the same coding conventions and security rules. Propose how to factor those shared rules into a versioned, installable plugin so each repo's own CLAUDE.md only needs repo-specific content, not a restated copy of the org-wide rules."

19. **Why, token-optimization angle:** At the largest scale, even "index + scoped files + skills" isn't enough if every session still re-explores the directory tree from scratch. Pairing this structure with Supermemory-style persistent memory (step 6) means the *navigation* itself — "where do I even look for X" — gets remembered across sessions, not just subsystem-specific facts.
    > "As this repo keeps growing, make sure Supermemory is also capturing navigation shortcuts — e.g. 'networking code lives across src/libraries/System.Net and src/native/networking, not just one place' — so future sessions don't have to rediscover the map itself, not just the details within it."

**What to point out:**
- Giant single CLAUDE.md = Claude "reads" it on every task whether relevant or not — wasted tokens every time.
- Scoped CLAUDE.md files + skills = cost proportional to where the work actually is.
- Supermemory adds a second axis: not just "what's true about the code" (static, in CLAUDE.md/skills) but "what have we learned by working in it" (dynamic, accumulated over time).
- ref-tools-mcp applies the same "load only what's needed" principle to *external* library/API docs, not just your own codebase.
- Repomix is the deliberate exception that proves the rule: fine for a scoped, one-off external hand-off; wrong as Claude's everyday working context at full-repo scale.
- `.claudeignore` and MCP server discipline are the same idea applied to file scanning and tool access respectively: don't load what you don't need.
- Dispatching `architect`/`refactor-cleaner` per-subsystem, on demand, keeps the main session's context from accumulating every subsystem it's ever glanced at — the subagent pays the exploration cost, not your main thread.
- A SessionStart hook automates "load only the relevant scoped file" so nobody has to remember to ask for it — same deterministic-over-remembered principle as the hooks in Use Cases 1 and 3.
- Growth doesn't stop at "split one CLAUDE.md" — the root index itself needs a discipline to stay small, staleness needs a hook to catch it, org-wide conventions need to move into a shared versioned plugin instead of being copy-pasted per repo, and navigation knowledge (not just facts) belongs in persistent memory.
- The pattern repeats at every scale: define what's cheap-and-always-loaded vs. expensive-and-loaded-on-demand, and keep re-drawing that line as the codebase (or the org) grows — it's never a one-time fix.

---

## Use Case 3 — Cross-repo change awareness without wasting tokens

**Real repos for this demo** (two genuinely related public .NET repos):
- `https://github.com/dotnet/aspnetcore` — the ASP.NET Core framework itself (defines the actual public APIs).
- `https://github.com/dotnet/AspNetCore.Docs` — the official docs repo documenting those APIs.

**Problem statement:** A change in `aspnetcore` (e.g. to a public API's behavior) sometimes requires a corresponding update in `AspNetCore.Docs`, but Claude shouldn't blindly scan the entire docs repo on every single edit. And once a fix spans both a code change and a UI-facing behavior change, we also want automated proof it actually works — not just an updated doc.

**Prompts to run, in order:**

1. Terminal setup (not a Claude prompt):
   `git clone --depth 1 https://github.com/dotnet/aspnetcore && git clone --depth 1 https://github.com/dotnet/AspNetCore.Docs`

2. **Why:** Keep a short, cheap-to-read pointer list rather than requiring Claude to know the entire docs repo's contents up front.
   > "In the aspnetcore repo, create a cross-repo trigger manifest at .claude/cross-repo.json. It should map a few real public API surface areas (e.g. files under src/Http or src/Mvc) to a note pointing at where the corresponding content likely lives in ../AspNetCore.Docs — just the trigger condition and a pointer, not full detail about the docs repo."

2b. **Why:** Rather than a generic prompt figuring out what changed and how to phrase the doc update each time, `everything-claude-code`'s `doc-updater` subagent is purpose-built for exactly this job — it stays scoped to "sync docs to match a code change" and nothing else, so it doesn't wander into unrelated refactoring while it's in there.
    > "Add the doc-updater subagent from everything-claude-code to this project. When triggered by the cross-repo hook, it should propose the specific doc edit needed in AspNetCore.Docs, not just flag that one is needed."

3. **Why:** A pinned note only works if it's actually enforced at the right moment — a hook does that automatically and every time, unlike relying on Claude to remember. This follows the same hook-based automation pattern `everything-claude-code` uses for its own memory-persistence and session-lifecycle hooks.
   > "Now create a PostToolUse hook in settings.json for the aspnetcore repo. It should fire when a file matching one of the trigger patterns in cross-repo.json is edited, print a warning telling me to check the corresponding docs in AspNetCore.Docs, and append a line to a shared CHANGELOG.md noting the cross-repo-relevant change."

4. Demo: edit a file under one of the trigger paths in `aspnetcore` — hook fires, warning shown, CHANGELOG updated.

5. Demo: edit an unrelated file in `aspnetcore` (outside the trigger list) — hook does *not* fire.

6. **Why:** When the cross-repo change also affects user-facing behavior (not just docs), don't just trust that the fix "should work" — prove it the same way Use Case 1 proved the pricing logic: with an automated test. Reuse `/webapp-testing` here to close the loop end-to-end.
   > "Using the webapp-testing skill, write a Playwright test against the locally running docs site (or sample app) that verifies the behavior described in the updated documentation actually matches the real API response."

7. **Why:** In real teams, the "other side" that needs to know about a change isn't always a git repo — sometimes it's an internal script, a wiki generator, or some other tool with no CLI or API a coding agent can call. **CLI-Anything** makes arbitrary software agent-native/callable-by-CLI, so Claude can trigger it as one more step in the same hook instead of that step being a manual, easily-forgotten task.
   > "Suppose our internal docs-linter tool has no CLI today. Show how CLI-Anything would expose it as an agent-callable command, and how our cross-repo hook could call it automatically after a trigger-matched edit."

8. **Why:** Many real engineering-to-docs handoffs don't live in CHANGELOG.md at all — they live in a shared Google Doc or Sheet the docs team actually watches. The **Google Workspace CLI** (with built-in AI agent skills for Drive/Docs/Sheets) lets the same hook write the cross-repo change note straight into that shared doc instead of (or in addition to) the repo's CHANGELOG — a more realistic destination for many teams.
   > "Instead of (or in addition to) CHANGELOG.md, use the Google Workspace CLI to append the same cross-repo change note as a new row in a shared Google Sheet the docs team already watches."

9. **Why:** A PostToolUse hook only fires *after* the edit already happened — fine for a warning, too late to prevent a genuinely dangerous cross-repo break (e.g. removing a public API a consumer still depends on). A **PreToolUse** hook can inspect the proposed edit against cross-repo.json *before* it's written, and block or require confirmation on a high-risk match — deterministic gatekeeping instead of after-the-fact cleanup.
   > "Add a PreToolUse hook that checks a proposed edit against cross-repo.json before it's applied. If the file matches a trigger marked 'breaking' in the manifest, require explicit confirmation before the edit proceeds instead of just warning afterward."

10. **Why, token-optimization angle:** Don't keep a cross-repo-checking subagent loaded in every session on the off chance a trigger fires — that's standing context cost paid on every single turn, matching-file or not. Instead, dispatch a narrow, single-purpose subagent *only* when the hook actually detects a trigger match — it gets a clean, isolated context scoped to just "check this one match," does its job, and returns, rather than the main session ballooning with repo-B exploration on every turn regardless of relevance.
    > "Instead of keeping full AspNetCore.Docs context loaded in this session, have the PostToolUse hook dispatch a lightweight subagent only when a trigger matches — that subagent alone reads AspNetCore.Docs, proposes the fix, and reports back a short summary, keeping my main session's context clean."

**What to point out:**
- The manifest is cheap to read and doesn't require knowing the other repo's whole contents.
- The hook is deterministic enforcement — it doesn't depend on Claude remembering under context pressure.
- Closing the loop with `/webapp-testing` turns "we updated the docs" into "we proved the docs are still true" — the same TDD-style proof discipline from Use Case 1, now applied across repos.
- CLI-Anything and the Google Workspace CLI both extend the same hook pattern beyond git repos — to internal tools and to shared documents non-engineers actually watch.
- doc-updater keeps the automated response scoped to "fix the doc," the same discipline Ponytail enforced for "don't over-build" in Use Case 1 — a narrow subagent beats a broad, unscoped prompt.
- PreToolUse vs PostToolUse is a choice between *preventing* a risky cross-repo break and merely *reacting* to one after the fact — pick based on how dangerous the trigger is.
- Lazy subagent dispatch (spawned only on an actual trigger match) is the token-optimization core of this whole use case: the cost of "knowing about" repo B stays near zero until there's a real reason to pay it.

---

## Verification checklist (run these before presenting live)
- Run `/init` on a shallow clone of `dotnet/runtime` to confirm the bloat problem before presenting it live.
- Install Ponytail and confirm `/ponytail full` changes Claude's behavior on a real prompt (less code, decision-ladder reasoning visible).
- Create and invoke the `reflect` skill in a scratch session to confirm it produces compressed, sensible output.
- Set up Supermemory locally and confirm a written memory can be queried back in a fresh session.
- Run a `/webapp-testing` Playwright test against a local demo page to confirm it works end-to-end (including Claude self-fixing a deliberately broken test or app).
- Install Superpowers and run one small feature through its full flow to confirm brainstorm → plan → subagent execution → TDD → review actually happens as described.
- Wire and trigger the cross-repo hook between `aspnetcore` and `AspNetCore.Docs` clones to confirm it fires only on matching paths.
- Confirm `strategic-compact` and `/checkpoint` behave as described (context visibly trimmed; session resumable without replay) before presenting them live.
