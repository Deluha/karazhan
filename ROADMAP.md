# Project Karazhan — Roadmap

The planned scope of the next release. `RELEASE_NOTES.md` records what shipped; this file records
what is planned. A release's section leaves this file when the release is tagged. Owner decisions are dated.
An item marked *confirmed* is a default the owner accepted on that date, including those that came from the
independent review of 2026-11-25; its exact form is still shown at the phase's sign-off.

## What Project Karazhan is

A local-first knowledge library, open source under the MIT license. It ingests the documents a person or a
team already has (Markdown, text, HTML, reStructuredText, Python docstrings, in local folders and git
repositories), indexes them into one SQLite file, and returns ranked, cited passages. It is reachable three
ways on one engine: a command line, a Python API, and an MCP server, so that AI coding agents can consult the
library instead of guessing.

What sets it apart from the retrieval projects that already exist:

- **One file, no services.** A library is a single SQLite database (a *tower*). No server, no vector
  database, no account. `karazhan export` writes one self-contained file that moves the library.
- **Offline by default.** Indexing and search make no network call. Nothing leaves the machine unless a
  remote source or a model download is switched on explicitly.
- **Every result is cited.** A passage always carries its document, heading path, line range and the version
  of the content it was read from, so a person or an agent can check it at the source.
- **Retrieval quality is measured, not asserted.** A built-in evaluation harness scores retrieval against
  golden questions. CI fails a change that makes the offline scores significantly worse, and a retrieval
  change states its scores with the real embedding model before it merges.

## Name and homage

The project is **Project Karazhan**; its repository name is `project-karazhan`, and its package and command
are `karazhan` (owner, 2026-11-25; the names `karazhan` and `project-karazhan` were unregistered on PyPI on
that date, and a name is only held once a real release is published under it).

The notice below stands in `README.md` and is carried by every release (owner, 2026-11-25):

> Project Karazhan is an independent open source project, named after the legendary raid as an homage. It is
> not affiliated with or endorsed by Blizzard Entertainment, and it has nothing to do with World of Warcraft.

The project uses no Blizzard artwork, logo, font or text.

## Codenames

Each system is named after a figure or encounter of Karazhan, the tower of the Guardian Medivh. A *tower* is
one library: one SQLite file plus its configuration.

| Codename | System | Why the name |
|---|---|---|
| **Medivh** | The lead feature: the query engine. It builds the query, runs the lexical and vector searches, fuses and shapes their results, and returns cited passages | The Guardian, master of the tower: every question goes to him |
| **The Curator** | Ingestion: sources, parsers, chunking, ignore rules, change detection, incremental sync | The construct that keeps the tower's collection |
| **Moroes** | The store: the SQLite schema, migrations, configuration and housekeeping. The only code that writes to a tower | The tower's steward |
| **Attumen** | Lexical search: the full-text index and BM25 ranking | The Huntsman: he tracks exact terms |
| **Shade of Aran** | Embeddings and vector search | The shade in the Guardian's Library: each passage's vector is its shade |
| **Chess Event** | Evaluation: golden question sets, retrieval metrics, the regression gate | A game with fixed rules and a scored outcome |
| **Maiden of Virtue** | Hygiene: secret redaction at ingestion, purge, the read-only guarantee towards sources | She purges what must not be kept |
| **Netherspite** | The MCP server: the portal through which agents reach the tower | The dragon of the portals |
| **Opera** | The command line: every command and how its output is rendered | The stage on which the others perform |

Where a codename appears (*confirmed 2026-11-25*): in the docs, in internal module names, and in a banner that an
interactive command prints to stderr. Never in a public identifier: command names, flags, configuration
keys, the Python API, MCP tool names and output fields use plain words (`search`, `sync`, `serve`), so that
scripts and agents never depend on a codename.

Reserved for later releases (see *Not in v0.1*): **Nightbane** (answer synthesis by a language model,
summoned on demand), **Terestian Illhoof** (the web crawler and its fetchers), **Prince Malchezaar**
(a shared, multi-user server with access control), **Netherspite's HTTP portal**, **the Opera's second act**
(a web interface).

## v0.1 — The Guardian's Library

Goal: a person points Karazhan at folders and repositories, and they or their AI agent get the right passage,
with its citation, in well under a second, entirely on their own machine.
Constraint: every CPython version from 3.11 that is not end-of-life, one SQLite file per tower, no service
to run.

### Scope rules (set 2026-11-25; the owner delegated the scope to the planning session)

- **Local and offline.** No network call at import, index, search or test time. A component that needs the
  network (a model download, a public evaluation set) runs only on an explicit command, and what it
  downloads is pinned to a revision and verified by checksum.
- **Retrieval only.** v0.1 returns ranked, cited passages. It generates no text and calls no language model.
- **Every result is cited.** No surface returns a passage without its citation. Retrieved text is data: no
  component executes it or follows instructions found in it, and no surface serves text that was not stored
  in the tower.
- **Read-only towards sources.** Karazhan never writes to, moves or deletes a source file or repository.
- **One engine, every surface.** A retrieval capability is implemented once and exposed by the Python API
  and the command line in the same pull request, and by the MCP server once Phase 10 has shipped.
- **Measured retrieval.** From Phase 7 on, a change to query building, ranking, chunking or embedding states
  its Chess Event scores before and after in the pull request. A significant drop needs the owner's sign-off.
  A default is tuned on the bundled set only with held-out evidence that it generalises.
- **Reproducible index.** A tower records what built it (parser, chunker, tokenizer, redaction ruleset,
  embedding model revision and settings). A change to one of them marks stale only what it invalidates.
- **Few dependencies.** The core installs with the standard library plus the few packages a phase's sign-off
  confirms. Embeddings and the MCP server are optional extras.
- **Stable contracts.** What is public is written down (Phase 1) and changes only with a release-notes entry.
  Machine-readable output carries a schema version. A newer Karazhan always opens an older tower.
- **Delivery.** One pull request per phase, several commits each. Database schema changes ship as numbered,
  forward-only migrations that are never edited once merged.
- **Sign-off.** Each phase lists what the owner confirms before its code is written.
- **Naming.** New systems take a Karazhan codename from the table above; a new codename is an owner decision.

### Design envelope (owner, 2026-11-25)

Measured on reference hardware named at the Phase 3 sign-off, by a CI check on a generated tower from
Phase 6 on, not first at release.

| Measure | Target |
|---|---|
| Tower size | up to 100,000 passages (roughly 10,000 documents) on a laptop |
| Lexical search, 95th percentile, warm process | under 100 ms at the size above |
| Hybrid search, 95th percentile, warm process (Python API, MCP server) | under 500 ms at the size above, exact vector search, no approximate index |
| One-shot command (`karazhan search`, cold start) | set at the Phase 8 sign-off from measurement; it pays interpreter start, model load and vector load |
| Indexing throughput with embeddings | set at the Phase 8 sign-off; embedding is resumable after an interruption |
| Re-sync of an unchanged source | no document re-parsed, no passage re-embedded; a renamed file and an unchanged passage of an edited file are not re-embedded either |
| Platforms | Linux, macOS and Windows, tested in CI from Phase 1 |

### Status

| Phase | Status | Pull request |
|---|---|---|
| 1 — Foundations: engineering | Not started | |
| 2 — Foundations: agent guidance | Not started | |
| 3 — Moroes: the store | Not started | |
| 4 — The Curator: ingestion | Not started | |
| 5 — Maiden of Virtue: hygiene | Not started | |
| 6 — Attumen: lexical search | Not started | |
| 7 — Chess Event: evaluation | Not started | |
| 8 — Shade of Aran: embeddings and vector search | Not started | |
| 9 — Medivh: the query engine | Not started | |
| 10 — Netherspite: the MCP server | Not started | |
| 11 — Release v0.1.0 | Not started | |

### Owner decisions from the review (owner, 2026-11-25)

All six recommendations were accepted. The phase named applies each one.

| # | Decision | Decided | Phase |
|---|---|---|---|
| O1 | The notice says the project "has nothing to do with World of Warcraft" while it is named after one of its raids; a reader can take the two as contradictory | The owner's wording is kept as it stands in *Name and homage*. The owner checks the trademark registers before the first release; this file is not legal advice | 1 |
| O2 | `AGENTS.md` as the tool-neutral contract for every coding agent, with `CLAUDE.md` importing it and adding the Claude-only parts | Yes | 2 |
| O3 | An alpha release `0.1.0a1` on PyPI once lexical search works (after Phase 6), which also holds the package name with a real release | Yes; an empty placeholder is not published | 6 |
| O4 | A CI job that downloads the pinned embedding model to score the real retriever, on a schedule and on a label | Yes; it is the only way CI can score the vector and hybrid modes | 8 |
| O5 | Language scope of the default tokenizer and embedding model | English first; other languages work lexically where the tokenizer allows, CJK is a documented limitation | 6, 8 |
| O6 | `RELEASE_NOTES.md` or `CHANGELOG.md` | Keep `RELEASE_NOTES.md`, written in the Keep a Changelog form and linked as the package's `Changelog` URL | 1 |

### Running a phase

One development session per phase, in order. A phase starts only when the previous phase's pull request
is merged. Kick-off prompt: "Implement Phase N of `ROADMAP.md` following its Running a phase section."

1. **Read.** This file, the contract (`AGENTS.md` / `CLAUDE.md`) and the development skill; route each commit
   through the skill's workflow file. Phases 1 and 2 create those files, so they read this file only.
2. **Sign-off first.** Present the phase's sign-off items to the owner as concrete proposals (schemas with
   sample rows, signatures, command lines with sample output, thresholds) and wait for confirmation before
   writing code. Record each confirmed decision under the phase, dated, in the phase's first commit.
3. **Check what moved.** Before proposing a dependency, a model, a protocol revision or a tool version, look
   up its current state; this file names what was current on 2026-11-25 and may be out of date.
4. **Implement.** One commit per table row, in order, each passing `python tools/verify.py` (in Phase 1,
   from the commit that adds it).
5. **Review.** The `reviewer` agent reviews the whole phase diff before the pull request opens; its
   findings are fixed in further commits and reviewed again.
6. **Deliver.** One pull request for the phase; the phase's Status row names it ("In review"). The next
   phase's first commit sets the previous row to "Merged". Each phase adds its entry to one
   "Unreleased" section at the top of `RELEASE_NOTES.md`; Phase 11 turns it into the release
   entry and removes the v0.1 section from this file.
7. **Stay in scope.** The scope rules bind every phase. A phase that finds an agreed item infeasible, or
   needs something not listed here, stops and asks the owner; it never narrows or widens scope itself.
8. **Lessons.** A review finding or a repeated mistake is fixed twice: in the code, and in the cheapest guard
   that stops it recurring (a lint rule, a checker rule or a test, then a hook or permission, then one
   guidance line). A guard for a rule arrives with the code it guards, not before.

### Phase 1 — Foundations: engineering

Builds no product feature. Everything here is independent of any coding agent.

| Commit | Content |
|---|---|
| 1 | Package skeleton in the `src/karazhan/` layout: `pyproject.toml` (single `version`, SPDX `license = "MIT"` with `license-files`, project URLs, dev tools as dependency groups, not a published extra), a committed lockfile for the development environment, `py.typed`, a `karazhan --version` command, `tests/` with one test, `.gitignore`, `.editorconfig`, `.devcontainer/devcontainer.json`, `tools/setup_dev_env.py` as the one documented set-up path. |
| 2 | Lint, format and types: `ruff check` and `ruff format`; a strict type check that gates from this commit on. Mechanical rules use an existing lint rule where one exists (no `print`, banned imports) rather than a custom checker. |
| 3 | Tests: `pytest` with a coverage floor, Hypothesis available for property tests, and a fixture that makes any network connection raise, so "offline" is tested, not promised. |
| 4 | `tools/verify.py`: lint, format check, type check, tests, one pass/fail table. Later phases add their steps to it. |
| 5 | `.pre-commit-config.yaml` (secret scan of staged changes, ruff, a fast `pytest` at pre-push) and `.gitleaks.toml`. |
| 6 | CI under `.github/workflows/`: `ci-tests` on Linux, macOS and Windows and on every supported Python version, plus one job on the lowest declared dependency versions and one on the oldest supported SQLite; every job prints its SQLite version and compile options. `ci-pre-commit` (every hook on the whole tree, then a pinned, checksum-verified secret scan of the commits and the tree). Workflow linting with `actionlint` and a workflow security audit, both gating. All of it ends in one `CI gate` job to require. Hardening: third-party actions pinned to a full commit SHA, read-only default token permissions, no credential left in the checkout, no `pull_request_target`. |
| 7 | Release pipeline, rehearsed now and not first at release: a build job (sdist and wheel, metadata check, install-from-wheel smoke test of the bare core) and a separate publish job that alone holds the publishing permission, runs in a protected environment the owner approves, uses PyPI trusted publishing (no stored token) and publishes with attestations. Rehearsed against TestPyPI. |
| 8 | `.github/dependabot.yml` (Python dependencies and GitHub Actions, weekly, with a cooldown), `.github/CODEOWNERS`, `.github/pull_request_template.md`, issue forms (bug, feature, a link to private security reporting). |
| 9 | Permanent docs: `README.md` (what it is, codenames, install, quick start, documentation layers, the homage notice verbatim), `CONTRIBUTING.md` (set-up, QA gates, branch and pull request flow, commit messages, contributions are under the MIT license), `CODE_OF_CONDUCT.md`, `SECURITY.md` (private reporting, supported versions), `RELEASE_NOTES.md` with the "Unreleased" section, and the stability policy below. |

Stability policy (*confirmed 2026-11-25*): public is what `karazhan` exports in `__all__`, the commands and flags, the
configuration keys, the exit codes, the `--json` output and the MCP tools; everything else is private. While
the version is 0.x a minor release may break a public item, with a release-notes entry. Machine-readable
output carries `schema_version`. A newer Karazhan opens and migrates any older tower; there is no downgrade.

Owner checklist (repository settings, done by the owner): rename the repository to `project-karazhan`;
turn on secret scanning with push protection, private vulnerability reporting and code scanning; a ruleset on
`main` requiring `CI gate` (no required approval while there is one maintainer); the PyPI and TestPyPI
trusted publishers and the protected environment.

Sign-off: the build backend; the type checker; the coverage floor; the supported Python versions
and the minimum SQLite version; the command-line library (*confirmed 2026-11-25:* `click`); the minimum
Python version (*confirmed 2026-11-25:* 3.11); the stability policy.

### Phase 2 — Foundations: agent guidance

Installs the process every later phase follows. It guards only what exists: a rule about the store or the
registries gets its mechanical guard in the phase that creates them.

| Commit | Content |
|---|---|
| 1 | The contract: `AGENTS.md` holds the operating principles, the workflow for any change, the hard rules in short form and the ask-first list; `CLAUDE.md` imports it and adds the Claude-only parts (O2). At most 80 lines and 8 KiB each. |
| 2 | The development skill `.claude/skills/project-karazhan-development/`: `SKILL.md` is a routing table that names the ONE file holding each task's detail. References: `conventions.md` (full form of each hard rule, simplicity audit, error handling), `style.md` (comments, console messages, CLI flags and exit codes), `architecture.md` (module map, codenames, naming), `qa-and-ci.md` (gates, the verify command, workflows, dependency policy), `checkers.md` (what each guard parses), `recipes.md` (grep-before-coding, the Implementation Decision format, orchestration, the lessons loop), and `workflows/changing-guidance.md`. Later phases add one workflow file and one routing row each. `evals/evals.json` holds routing and ask-first prompts to run by hand before a guidance change merges. |
| 3 | Agents under `.claude/agents/`: `implementer` (writer, from a complete brief), `reader` (read-only, for reads over three files), `reviewer` (independent diff review, never the author, never edits), `scribe` (notes outside the repository, never committed). Model tier and effort live only in the agent frontmatter and one roles table in `recipes.md`, as tier aliases, never dated model identifiers. |
| 4 | Checkers under `tools/`, each with its tests and its step in `tools/verify.py`: `check_guidance_integrity.py` (pointers and anchors resolve, the contract's budget, one routing row per reference, pointer-only rules and command skills) and `check_docs_sync.py` (a code change in a documented area moves its `README.md`; `[skip-docs]` on its own line in the commit body is the escape hatch; it runs in CI against the pull request base as `ci-docs-sync`, and locally through the verify). `check_conventions.py` starts with the rules no lint rule covers and grows with the lessons loop. |
| 5 | Hooks and settings: `.claude/settings.json` (allow the verify, checker, `pytest`, `ruff` and read-only `git` commands; `ask` on the guidance machinery and CI paths; deny reads of `.env`, key and certificate files), `.claude/hooks/session-start.sh` (dev set-up in cloud sessions through `tools/setup_dev_env.py`, skipped when nothing that decides the install changed), `.claude/hooks/post_edit_check.py` (lint and the convention checker after a Python edit; never blocks on its own error). |
| 6 | Path rules `.claude/rules/` and command skills: `/verify`, `/review-diff`, `/guidance-change`. Both kinds stay pointer-only: they name the workflow file and its ask-first gate, nothing more. The pull request template gains its process sections. Docs and release-notes entry. |

What the guidance says (owner, 2026-11-25; the final wording is shown at sign-off):

- **Operating principles.** (1) Flag uncertainties: ask, do not assume; validate every request against the
  repository's patterns and flag a conflict rather than silently comply; unattended, take the most reasonable
  reading and record the assumption. (2) The orchestrator designs and signs off, the writer implements, the
  reader reads, the reviewer reviews; the orchestrator makes only small edits itself (one file, about 20
  changed lines at most, no ask-first item, no new logic). (3) Be concise: lead with the outcome, and never
  drop the confidence label on an architectural call, the flag on a pattern conflict, or the better approach
  when one exists. (4) Modular, single source of truth: one canonical definition per helper and constant;
  few surgical changes over sweeping refactors.
- **Workflow for any change.** Understand the request → route through the skill and read the routed file →
  grep the established pattern → validate against it and flag conflicts → implement → verify → update the
  changed area's docs → simplicity audit and the `reviewer` agent on every non-trivial diff.
- **Hard rules.**
  1. Registries are imported, never hardcoded: sources, parsers and embedders are looked up in their one
     registry.
  2. Shared helpers are reused, never re-implemented. The engine never prints: it logs through `logging` and
     returns values; only the Opera renders. Standard output carries the result and nothing else; banners,
     progress and diagnostics go to standard error.
  3. Offline by default: no network call at import, index, search or test time.
  4. No files the user did not ask for: `tests/` is the only test location, no markdown beyond the
     permanent docs.
  5. Moroes is the only writer: every SQLite statement lives in the store module, and a schema change is a
     new numbered migration.
  6. A retrieval change carries its Chess Event scores (binding from Phase 7).
  7. Docs move with the code.
  8. Entry points hold only their flow: a command parses, calls the engine and renders; logic lives in
     private `_*.py` engine modules behind the package facade.
  9. One engine, every surface: a capability ships on the Python API and the command line together, and on
     the MCP server once it exists.
- **Ask first.** The schema of a merged migration and any destructive step; query building, ranking, fusion
  and chunking defaults; the embedding model default; anything that sends or downloads data; the citation
  format; anything the stability policy calls public; a new dependency; a new codename; redaction rules; a
  Chess Event threshold; the guidance machinery and CI (`AGENTS.md`, `CLAUDE.md`, `.claude/settings.json`,
  `.claude/agents/`, `.claude/hooks/`, `.github/workflows/`, `.pre-commit-config.yaml`, `.gitleaks.toml`).
- **Pull request template sections.** Summary; Changes; Docs updated; QA gates run; Review agents run; Not
  tested live / to do by hand; Lessons (each finding and the guard added); Schema changes; Retrieval scores
  (Chess Event, before and after); Ask-first items touched.
- **Commit messages.** Imperative subject of at most 72 characters; the body says why when it is not obvious.

Sign-off: the hard rules and the ask-first list as worded above; the agents' tiers and tool lists; the
contract's budget.

### Phase 3 — Moroes: the store

| Commit | Content |
|---|---|
| 1 | Tower layout and configuration: where a tower lives (*confirmed 2026-11-25:* `.karazhan/tower.db` and `.karazhan/config.toml` in the directory where `karazhan init` ran, overridable by `--tower` and `KARAZHAN_TOWER`); found from a subdirectory by walking up, as git does (*confirmed 2026-11-25*). `init` creates the tower readable by its owner only and writes a `.karazhan/.gitignore`, so an index of private files is not committed by accident. Configuration precedence: flag, environment, file, default. |
| 2 | Schema and migration runner: `sources`, `documents`, `passages`, `meta`; numbered, forward-only SQL migrations, one transaction each. A tower carries an application id and a schema version: a foreign SQLite file and a newer schema are refused. A backup is taken through SQLite's own backup mechanism before a migration. A test migrates a committed fixture tower from every past schema version. |
| 3 | The store API: the one module that opens connections and runs statements (foreign keys on, a busy timeout, one writer at a time with a clear message for the second). Write-ahead logging with a checkpoint at the end of every write command; a fallback journal mode where the location does not support it (network drives). The build fingerprint in `meta` (scope rule "Reproducible index"). A start-up check that the SQLite build has FTS5 and the minimum version, with a clear message when it has not. The lint ban on SQLite access outside the store (hard rule 5). |
| 4 | Commands `karazhan init`, `karazhan status` (sources, counts, schema version, size, fingerprint and what is stale), `karazhan doctor` (integrity check, orphan rows, SQLite version and compile options, location checks) and `karazhan export` (one self-contained, compacted file: the supported way to copy a tower). Python API equivalents. The exit-code table. |
| 5 | The workflow file `workflows/schema-changes.md`, its routing row, docs and release-notes entry. |

Sign-off: the tower location, discovery and override order; the four tables' columns and keys; the migration
file naming; the fingerprint's fields; the design envelope and its reference hardware; the exit codes
(*confirmed 2026-11-25:* 0 success, 1 failure, 2 usage error, a distinct code for "tower needs migration"); the output of
`status` and `doctor`.

### Phase 4 — The Curator: ingestion

| Commit | Content |
|---|---|
| 1 | Source registry and the two v0.1 sources: a local directory and a git repository already cloned on disk (tracked files at the checked-out commit; the commit and a dirty-worktree flag are recorded). `karazhan source add / list / remove`. |
| 2 | Ignore rules, in the first sync and not later: `.gitignore` honoured, a `.karazhanignore` file, built-in exclusions (binary files, dependency and build directories, `.env`, key and certificate files), a size limit, and no symbolic link followed out of the source root. |
| 3 | Parser registry and the v0.1 parsers: Markdown, plain text, reStructuredText, HTML (main text, headings kept), and docstrings and comments of Python files through the standard library. Each returns a heading tree with line ranges. Encoding detection with a reported fallback. |
| 4 | Chunking: heading-aware passages with a size target and overlap, each carrying its heading path and line range; a passage never crosses a top-level heading and never splits a fenced code block, a table or a list item. The chunker has a version, recorded in the fingerprint. |
| 5 | `karazhan sync`: change detection by content hash at document and passage level. An unchanged document is not re-parsed; a renamed document is re-pointed; documents gone from the source are removed. Each document is written in one transaction, so an interrupted sync leaves a consistent tower and resumes. A summary of added, changed, renamed, removed and skipped. |
| 6 | Failure handling: an unreadable or oversized file is reported and skipped, never fatal; a `--dry-run` that writes nothing. Property tests: every passage's line range lies inside its document and reproduces its text; a second sync changes nothing. A test that kills a sync midway. |
| 7 | The workflow files `workflows/adding-a-source.md` and `workflows/adding-a-parser.md`, routing rows, docs and release-notes entry. |

Sign-off: the source and parser protocols, the file types and size limit, the built-in exclusion list, the
unit of the chunk size target (it must fit the embedding model's input limit chosen in Phase 8, so it is
provisional until then), the overlap, the passage identity across syncs, the `sync` summary layout.

### Phase 5 — Maiden of Virtue: hygiene

Runs before anything is indexed or scored on purpose: redaction changes passage text, so it must be in place
before the first index and the first baseline exist.

| Commit | Content |
|---|---|
| 1 | Secret redaction at ingestion: text matching credential patterns from a maintained, versioned ruleset is replaced before it is stored, including multi-line private-key blocks. Redaction keeps the line count, so citations stay true. Matches are counted in the `sync` summary; the ruleset version is in the fingerprint. |
| 2 | `karazhan purge` (remove a source, a path or everything matching a pattern), which also clears the removed text from the file itself (deleted pages, the write-ahead log, later the search index and vectors), not only from query results. |
| 3 | The read-only guarantee towards sources, backed by a test that runs every command against a read-only source. |
| 4 | The threat model in `SECURITY.md`: the tower is a plaintext copy of what was indexed; redaction is best-effort pattern matching and not a security boundary; what `purge` does and does not guarantee. The "what Karazhan stores and what it never does" section of the README. Release-notes entry. |

Sign-off: the ruleset and where it comes from, the replacement text, whether an entropy rule is used (decided
on measured false positives over documentation text), the behaviour on a match (*confirmed 2026-11-25:*
redact and continue, not skip the file), the `purge` confirmation prompt.

### Phase 6 — Attumen: lexical search

The tokenizer and the column weights shipped here are provisional: Phase 7 sweeps them with evidence.

| Commit | Content |
|---|---|
| 1 | The FTS5 index over passages (text, heading path, document title, file path), kept in step by `sync` in the same transaction; the tokenizer configuration recorded in the fingerprint; migration. |
| 2 | Query building: a natural-language question must not need every word in one passage. Terms are combined so that ranking, not exclusion, decides (*confirmed 2026-11-25:* all terms first, any term as the fallback); quoted phrases and prefixes are honoured only when typed; user input is escaped so it can never break the query syntax, with a property test over arbitrary input. BM25 ranking with column weights. |
| 3 | The result model shared by every later phase: passage text, score, source, document path, heading path, line range, and the version of the content (content hash, sync time, the commit for a git source), so a surface can say "the source changed since the last sync". Filters by source, path glob and file type. |
| 4 | `karazhan search "<query>"` with `--limit`, the filters and `--json`; the Python API `Tower.search`. Human output shows the snippet with its citation. Banner and progress on standard error only. |
| 5 | The latency check: a generated 100,000-passage tower, checked in CI against the envelope with a loose margin. Docs and release-notes entry. After the merge the owner tags `v0.1.0a1` (O3). |

Sign-off: the provisional tokenizer and column weights; the query-building rule; the result
model's fields and the citation format (ask-first from here on); the `search` flags and both output layouts.

### Phase 7 — Chess Event: evaluation

Built before the vector and hybrid phases on purpose: Attumen's scores become the baseline that Shade of Aran
and Medivh must beat.

| Commit | Content |
|---|---|
| 1 | The bundled evaluation set under `tests/`, excluded from the published package: a permissively licensed document set and golden questions with the passages that answer them, in strata (exact identifier or keyword, natural-language question, paraphrase with no shared words, answer spread over several passages, no answer in the corpus). `THIRD_PARTY_NOTICES.md` records its licence. |
| 2 | Metrics: recall at k, mean reciprocal rank, nDCG at k, per question, per stratum and aggregated; a retriever protocol so any search mode can be scored. A comparison reports paired per-question differences with a bootstrap confidence interval. |
| 3 | `karazhan eval` against the bundled set, a user's own golden file (documented format), or a public benchmark set fetched by an explicit `karazhan eval pull` and never bundled. Every score carries the set's version and the tower's fingerprint. A table and `--json`. |
| 4 | The offline regression gate: the committed baseline, and a step in `tools/verify.py` that fails when a mode is significantly worse than its baseline, not on a raw difference. |
| 5 | The lexical sweep: tokenizer options, an indexed column of split identifier parts, column weights and the query-building variants, scored on the bundled set and on a held-out public set; the defaults of Phase 6 are confirmed or changed on that evidence. |
| 6 | The workflow file `workflows/retrieval-changes.md`, its routing row, hard rule 6 and the pull request template's Retrieval scores section made binding, docs and release-notes entry. |

Sign-off: the corpus and its licence; the size of the golden set (*confirmed 2026-11-25:* a few hundred questions) and
who writes it (questions drafted by a language model are reviewed by a person, and a subset shares no wording
with its passage); the metrics and the values of k; the significance rule; the held-out public sets; the
golden-file format.

### Phase 8 — Shade of Aran: embeddings and vector search

| Commit | Content |
|---|---|
| 1 | The embedder protocol and registry, with separate entry points for queries and for documents, because many models need a different prefix for each and a mix-up fails silently. A deterministic hashing embedder used by the test suite, so tests stay offline and reproducible. |
| 2 | The default local embedder as the optional extra `karazhan[embeddings]`. `karazhan model pull` downloads it explicitly, at a pinned revision, verified by checksum; nothing downloads implicitly. |
| 3 | What is embedded: document title, heading path and passage text together, so a passage carries its context. Passages longer than the model's input limit are counted and reported, never silently truncated. |
| 4 | Vector storage in the tower, in a layout that loads fast from one file, normalised at write time, tagged with the full model fingerprint. `sync` embeds only new or changed passages, in resumable batches; a fingerprint change marks vectors stale and `karazhan sync --reembed` rebuilds them. A long-lived process keeps the vectors in memory and reloads when the tower changes. |
| 5 | Exact cosine search over the stored vectors; `karazhan search --mode vector`; the Python API. Without the extra, lexical search keeps working, with one clear message. |
| 6 | The real-model evaluation job (O4): scores for the vector mode against the lexical baseline on the bundled and the held-out sets, recorded in the docs. The cold-start and indexing-throughput targets of the envelope are set from measurement. Docs and release-notes entry. |

Sign-off: the default model, chosen by Chess Event scores on the bundled and held-out sets among current
small CPU models, with its size, input limit, languages (O5), prefixes and the licence of its weights (the
session surveys the current options); its runtime (*confirmed 2026-11-25:* ONNX on the CPU, no PyTorch
dependency); the vector storage (*confirmed 2026-11-25:* float32 and exact search with `numpy`, no
approximate index in v0.1; packing and optional quantisation *confirmed 2026-11-25* as the direction, their form shown at sign-off); batch size; the chunk size target
of Phase 4 confirmed against the model's input limit.

### Phase 9 — Medivh: the query engine

The lead feature: one question in, the best cited passages out.

| Commit | Content |
|---|---|
| 1 | Hybrid retrieval: Attumen and Shade of Aran run on the same query and their rankings are fused (*confirmed 2026-11-25:* reciprocal rank fusion, with its standard constant and equal weights). Hybrid is the default mode when vectors exist; lexical is the fallback, also when one side returns nothing. |
| 2 | Result shaping: near-duplicate passages collapsed, adjacent passages of one document merged, a cap per document so one file cannot fill the list. |
| 3 | Context assembly: the top passages within a size budget, each with its citation, in the human layout, as `--json` and as prompt-ready Markdown. The Python API equivalent. |
| 4 | A reranker protocol behind the same registry pattern, with no reranking model bundled. |
| 5 | Chess Event scores for hybrid against both single modes on the bundled and the held-out sets; the committed baseline moves to hybrid. Docs and release-notes entry. |

Sign-off: the candidate depth per retriever; the duplicate and merge rules; the default budget and
per-document cap; the minimum hybrid scores to ship; and the shape of the command (*confirmed 2026-11-25:* one `search`
command that shapes and budgets by default, with no separate `ask`, since nothing is answered until
Nightbane exists and two near-identical tools make an agent choose badly). A fusion constant or weight
changes only with held-out evidence.

### Phase 10 — Netherspite: the MCP server

| Commit | Content |
|---|---|
| 1 | `karazhan serve` over stdio as the optional extra `karazhan[mcp]`, on the official MCP Python SDK. Read-only by construction: the connection cannot write, and the server sees later syncs. Standard output is the protocol channel and carries nothing else. |
| 2 | Tools on the existing engine: search, read a document section, list sources. Each is annotated as read-only, declares an output schema and returns typed fields (citation fields are fields, not prose) with a text fallback, under a size cap, with clear results for "no hits", "no vectors" and "stale index". |
| 3 | Safety: the section-read tool serves only text stored in the tower, addressed by passage or document id; it never opens a path from disk. Retrieved text never appears in a tool description, and each description says that passage content is untrusted document text. Tests for both, and a test of a sync running during serve. The threat model in `SECURITY.md` gains the agent-facing part: a passage can carry instructions aimed at the agent that reads it, which Karazhan can label but not remove. |
| 4 | Tests driving the server through an in-process client; a documented set-up for Claude Code and other MCP clients. |
| 5 | The workflow file `workflows/adding-an-mcp-tool.md`, its routing row, hard rule 9 extended to the server, docs and release-notes entry. |

Sign-off: the protocol revision targeted and the SDK major (on 2026-11-25 the `mcp` package on PyPI is at
2.3.0, after a breaking 2.0; the session confirms the current state); the tool names, descriptions,
annotations and schemas (ask-first from here on); the result size cap; whether `sync` is exposed as a tool
(*confirmed 2026-11-25:* no, the server stays read-only).

### Phase 11 — Release v0.1.0

| Commit | Content |
|---|---|
| 1 | End-to-end test on the bundled corpus: init, add a source, sync, search, eval, export, serve. It runs the README's quick-start commands themselves, so they cannot drift. Install-from-wheel smoke tests for the core and for each extra. |
| 2 | README completed: each codename's section, performance figures measured against the design envelope, known limitations (code bodies are not indexed; languages). |
| 3 | `RELEASE_NOTES.md`: the "Unreleased" section becomes the v0.1.0 entry; the v0.1 section leaves this file; `SECURITY.md` supported versions set; `THIRD_PARTY_NOTICES.md` complete (evaluation corpus, embedding model). |

Sign-off: the measured figures against the envelope, the tag. The release itself goes through the pipeline
rehearsed in Phase 1 and used for the alpha.

### Not in v0.1

- **Deferred, with a codename reserved:** Nightbane, answer synthesis by a language model, opt-in and summoned
  per call; Terestian Illhoof, crawling and fetching websites; Prince Malchezaar, a shared multi-user server
  with access control; Netherspite's HTTP portal; the Opera's second act, a web interface.
- **Deferred, retrieval quality (the first v0.2 candidates):** a bundled reranking model; context for a
  passage written by a language model; late chunking; multi-vector (late-interaction) retrieval; score-based
  fusion; structure-aware chunking of source code in several languages.
- **Deferred:** PDF and office documents; a watch mode that syncs on file change; an approximate or
  quantised vector index for towers beyond the design envelope; searching several towers at once; remote git
  cloning; MCP resources.
- **Out of scope:** a hosted service, telemetry of any kind, editing documents.

## After v0.1 — review before v0.2 is scoped

Starts when Phase 11 is merged and v0.1.0 is tagged; this section stays when the v0.1 section is removed.

- **Repository and codebase.** Every module against the hard rules, the module layout and the docs; dead
  code, duplicated helpers, stale docs and tests that no longer pin what they claim.
- **Retrieval.** The Chess Event scores re-run on a corpus neither the bundled nor the held-out sets cover,
  ideally a real user's, to check the defaults once more.
- **Guidance.** Each lesson recorded in the phases' pull requests against the guard it produced; prose that a
  checker now enforces is deleted and replaced by a pointer.

Read-only: findings are reported to the owner, ranked, with the evidence behind each; nothing is fixed
without sign-off. The v0.2 scope is agreed with the owner afterwards, starting from *Not in v0.1*.
