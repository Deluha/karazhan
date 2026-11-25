# Project Karazhan — Roadmap

The planned scope of the next release. `RELEASE_NOTES.md` records what shipped; this file records
what is planned. A release's section leaves this file when the release is tagged. Owner decisions are dated;
an item marked *proposed* is a default the owner has not confirmed yet.

## What Project Karazhan is

A local-first knowledge library, open source under the MIT license. It ingests the documents a person or a
team already has (Markdown, text, HTML, reStructuredText, source-code docs, local folders and git
repositories), indexes them into one SQLite file, and answers questions with ranked, cited passages. It is
reachable three ways on one engine: a command line, a Python API, and an MCP server, so that AI coding
agents can consult the library instead of guessing.

What sets it apart from the retrieval projects that already exist:

- **One file, no services.** A library is a single SQLite database (a *tower*). No server, no vector
  database, no account. Copy the file and the library moves with it.
- **Offline by default.** Indexing and search make no network call. Nothing leaves the machine unless a
  remote source or a hosted model is switched on explicitly.
- **Every result is cited.** A passage always carries its document, heading path and line range, so a person
  or an agent can check it at the source.
- **Retrieval quality is measured, not asserted.** A built-in evaluation harness scores every retrieval
  change against golden questions, and CI fails a change that makes the scores worse.

## Codenames

Each system is named after a figure or encounter of Karazhan, the tower of the Guardian Medivh, and its
entry points open with that name in their banner. A *tower* is one library: one SQLite file plus its
configuration.

| Codename | System | Why the name |
|---|---|---|
| **Medivh** | The lead feature: the query engine. It understands the question, runs the lexical and vector searches, fuses and reranks their results, and returns cited passages | The Guardian, master of the tower: every question goes to him |
| **The Curator** | Ingestion: sources, parsers, chunking, change detection, incremental sync | The construct that keeps the tower's collection |
| **Moroes** | The store: the SQLite schema, migrations, configuration and housekeeping. The only code that writes to a tower | The tower's steward |
| **Attumen** | Lexical search: the full-text index and BM25 ranking | The Huntsman: he tracks exact terms |
| **Shade of Aran** | Embeddings and vector search | The shade in the Guardian's Library: each passage's vector is its shade |
| **Chess Event** | Evaluation: golden question sets, retrieval metrics, the CI regression gate | A game with fixed rules and a scored outcome |
| **Maiden of Virtue** | Hygiene: ignore rules, secret redaction at ingestion, the read-only guarantee towards sources | She purges what must not be kept |
| **Netherspite** | The MCP server: the portal through which agents reach the tower | The dragon of the portals |
| **Opera** | The command line: every command and how its output is rendered | The stage on which the others perform |

Reserved for later releases (see *Not in v0.1*): **Nightbane** (answer synthesis by a language model,
summoned on demand), **Terestian Illhoof** (the web crawler and its fetchers), **Prince Malchezaar**
(a shared, multi-user server with access control), **Netherspite's HTTP portal**, **the Opera's second act**
(a web interface).

## v0.1 — The Guardian's Library

Goal: a person points Karazhan at folders and repositories, and they or their AI agent get the right passage,
with its citation, in well under a second, entirely on their own machine.
Constraint: Python 3.11 or later, one SQLite file per tower, no service to run.

### Scope rules (set 2026-11-25; the owner delegated the scope to the planning session)

- **Local and offline.** No network call at import, index, search or test time. A component that needs the
  network (a model download, a remote git clone) is opt-in, named in the configuration, and never the default.
- **Retrieval only.** v0.1 returns ranked, cited passages. It generates no text and calls no language model.
- **Every result is cited.** No surface returns a passage without its document path, heading path and line
  range. Retrieved text is data: no component executes it or follows instructions found in it.
- **Read-only towards sources.** Karazhan never writes to, moves or deletes a source file or repository.
- **One engine, every surface.** A retrieval capability is implemented once and exposed by the Python API
  and the command line in the same pull request, and by the MCP server once Phase 9 has shipped.
- **Measured retrieval.** From Phase 5 on, a change to ranking, chunking or embedding states its Chess Event
  scores before and after in the pull request. A score drop needs the owner's sign-off.
- **Few dependencies.** The core installs with the standard library plus the few packages a phase's sign-off
  confirms. Embeddings and the MCP server are optional extras.
- **Delivery.** One pull request per phase, several commits each. Database schema changes ship as numbered
  migrations that are never edited once merged.
- **Sign-off.** Each phase lists what the owner confirms before its code is written.
- **Naming.** New systems take a Karazhan codename from the table above; a new codename is an owner decision.

### Design envelope (proposed, confirmed at the Phase 2 sign-off)

| Measure | Target |
|---|---|
| Tower size | up to 100,000 passages (roughly 10,000 documents) on a laptop |
| Lexical search, 95th percentile | under 100 ms at the size above |
| Hybrid search, 95th percentile | under 500 ms at the size above, exact vector search, no approximate index |
| Re-sync of an unchanged source | no document re-parsed, no passage re-embedded |
| Platforms | Linux, macOS and Windows; CI runs the test suite on Linux, the release phase adds the other two |

### Status

| Phase | Status | Pull request |
|---|---|---|
| 1 — Foundations: the development process | Not started | |
| 2 — Moroes: the store | Not started | |
| 3 — The Curator: ingestion | Not started | |
| 4 — Attumen: lexical search | Not started | |
| 5 — Chess Event: evaluation | Not started | |
| 6 — Shade of Aran: embeddings and vector search | Not started | |
| 7 — Medivh: the query engine | Not started | |
| 8 — Maiden of Virtue: hygiene | Not started | |
| 9 — Netherspite: the MCP server | Not started | |
| 10 — Release v0.1.0 | Not started | |

### Running a phase

One development session per phase, in order. A phase starts only when the previous phase's pull request
is merged. Kick-off prompt: "Implement Phase N of `ROADMAP.md` following its Running a phase section."

1. **Read.** This file, `CLAUDE.md` and the development skill; route each commit through the skill's
   workflow file. Phase 1 creates those files, so it reads this file only.
2. **Sign-off first.** Present the phase's sign-off items to the owner as concrete proposals (schemas with
   sample rows, signatures, command lines with sample output, thresholds) and wait for confirmation before
   writing code. Record each confirmed decision under the phase, dated, in the phase's first commit.
3. **Implement.** One commit per table row, in order, each passing `python tools/verify.py`.
4. **Review.** The `reviewer` agent reviews the whole phase diff before the pull request opens; its
   findings are fixed in further commits and reviewed again.
5. **Deliver.** One pull request for the phase; the phase's Status row names it ("In review"). The next
   phase's first commit sets the previous row to "Merged". Each phase adds its entry to one
   "v0.1.0 — in development" section at the top of `RELEASE_NOTES.md`; Phase 10 turns it into the release
   entry and removes the v0.1 section from this file.
6. **Stay in scope.** The scope rules bind every phase. A phase that finds an agreed item infeasible, or
   needs something not listed here, stops and asks the owner; it never narrows or widens scope itself.
7. **Lessons.** A review finding or a repeated mistake is fixed twice: in the code, and in the cheapest guard
   that stops it recurring (a checker rule or a test, then a hook or permission, then one guidance line).

### Phase 1 — Foundations: the development process

Builds no product feature. It installs the process every later phase follows, so that the guidance, the
checks and the CI exist before the first line of product code.

| Commit | Content |
|---|---|
| 1 | Package skeleton: `pyproject.toml` (single `version`, `ruff` and `pytest` configuration, dev extra), the `karazhan/` package with a `karazhan --version` command, `tests/` with one test, `.gitignore`, `.editorconfig`, `.devcontainer/devcontainer.json`, `tools/setup_dev_env.py` (editable install, dev tools, git hooks). |
| 2 | `CLAUDE.md`: the always-loaded contract. Operating principles, the workflow for any change, the hard rules in short form, the ask-first list. At most 80 lines and 8 KiB. |
| 3 | The development skill `.claude/skills/project-karazhan-development/`: `SKILL.md` is a routing table that names the ONE file holding each task's detail. References: `conventions.md` (full form of each hard rule, simplicity audit, error handling), `style.md` (comments, console messages, CLI flags), `architecture.md` (module map, codenames, naming), `qa-and-ci.md` (gates, the verify command, workflows, README and requirements policy), `checkers.md` (what each guard parses), `recipes.md` (grep-before-coding, the Implementation Decision format, orchestration, the lessons loop), and `workflows/changing-guidance.md`. Later phases add one workflow file and one routing row each. `evals/evals.json` holds routing and ask-first prompts to run by hand before a guidance change merges. |
| 4 | Agents under `.claude/agents/`: `implementer` (writer, from a complete brief), `reader` (read-only, for reads over three files), `reviewer` (independent diff review, never the author, never edits), `scribe` (notes outside the repository, never committed). Model tier and effort live only in the agent frontmatter and one roles table in `recipes.md`. |
| 5 | Checkers under `tools/`: `check_conventions.py` (the mechanical hard rules, plus no tracked secret or build artifact), `check_guidance_integrity.py` (pointers and anchors resolve, the `CLAUDE.md` budget, one routing row per reference, pointer-only rules and command skills), `check_docs_sync.py` (a code change in a documented area moves its `README.md`; `[skip-docs]` on its own line in the commit body is the escape hatch), and `verify.py` (every checker, `ruff check .`, `pytest -q`, docs-sync against `origin/main`, one pass/fail table). Each checker has its tests. |
| 6 | Hooks and settings: `.claude/settings.json` (allow the verify, checker, `pytest`, `ruff` and read-only `git` commands; `ask` on the guidance machinery and CI paths; deny reads of `.env`, key and certificate files), `.claude/hooks/session-start.sh` (dev setup in cloud sessions, skipped when nothing that decides the install changed), `.claude/hooks/post_edit_check.py` (ruff and the convention checker after a Python edit; never blocks on its own error). |
| 7 | Path rules `.claude/rules/` and command skills: `/verify`, `/review-diff`, `/guidance-change`. Both kinds stay pointer-only: they name the workflow file and its ask-first gate, nothing more. |
| 8 | `.pre-commit-config.yaml` (gitleaks on staged changes, conventions, guidance integrity, ruff, docs-sync at the commit-msg stage, a fast `pytest` at pre-push) and `.gitleaks.toml`. |
| 9 | CI under `.github/workflows/`: `ci-pre-commit` (every hook on the whole tree, then a pinned, checksum-verified gitleaks scan of the commits and the tree), `ci-docs-sync` (against the pull request base), `ci-tests` (`pytest` with coverage, advisory `mypy`, gating `actionlint`, ending in one `CI gate` job to require). Third-party actions pinned to a full commit SHA. `.github/dependabot.yml` (pip and github-actions, weekly), `.github/CODEOWNERS`, `.github/pull_request_template.md`. |
| 10 | Permanent docs: `README.md` (what it is, codenames, quick start, documentation layers, release and deployment, a notice that the project is not affiliated with or endorsed by Blizzard Entertainment), `CONTRIBUTING.md` (setup, QA gates, branch and pull request flow, commit messages), `SECURITY.md` (private reporting, supported versions, no credential ever tracked, rotation), `RELEASE_NOTES.md` with the "v0.1.0 — in development" section. |

What the guidance says (proposed; the wording is confirmed at sign-off):

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
  2. Shared helpers are reused, never re-implemented; console output goes through the Opera's message
     helpers, never a raw `print`.
  3. Offline by default: no network call at import, index, search or test time.
  4. No files the user did not ask for: `tests/` is the only test location, no markdown beyond the
     permanent docs.
  5. Moroes is the only writer: every SQLite statement lives in the store module, and a schema change is a
     new numbered migration.
  6. A retrieval change carries its Chess Event scores (binding from Phase 5).
  7. Docs move with the code.
  8. Entry points hold only their flow: a command parses, calls the engine and renders; logic lives in
     private `_*.py` engine modules behind the package facade.
  9. One engine, every surface: a capability ships on the Python API and the command line together, and on
     the MCP server once it exists.
- **Ask first.** The schema of a merged migration and any destructive step; ranking, fusion and chunking
  defaults; the embedding model default; anything that sends data off the machine; the citation format;
  public signatures of the Python API, command names and flags, MCP tool names and schemas; a new dependency;
  a new codename; redaction rules; a Chess Event threshold; the guidance machinery and CI (`CLAUDE.md`,
  `.claude/settings.json`, `.claude/agents/`, `.claude/hooks/`, `.github/workflows/`,
  `.pre-commit-config.yaml`, `.gitleaks.toml`).
- **Pull request template sections.** Summary; Changes; Docs updated; QA gates run; Review agents run; Not
  tested live / to do by hand; Lessons (each finding and the guard added); Schema changes; Retrieval scores
  (Chess Event, before and after); Ask-first items touched.
- **Commit messages.** Imperative subject of at most 72 characters; the body says why when it is not obvious.

Sign-off: the hard rules and the ask-first list as worded above; the agents' model tiers and tool lists; the
`CLAUDE.md` budget; the package and command name (`karazhan`, subject to its availability on PyPI; the
repository is currently named `project-kazazhan`, to be renamed or kept by the owner); the command-line
library (*proposed:* `click`); the minimum Python version (*proposed:* 3.11).

### Phase 2 — Moroes: the store

| Commit | Content |
|---|---|
| 1 | Tower layout and configuration: where a tower lives (*proposed:* `.karazhan/tower.db` and `.karazhan/config.toml` in the directory where `karazhan init` ran, overridable by `--tower` and `KARAZHAN_TOWER`), configuration loading with defaults. |
| 2 | Schema and migration runner: `sources`, `documents`, `passages`, `meta`; numbered SQL migrations applied in order inside one transaction each; a tower records its schema version and refuses a newer one. |
| 3 | The store API: the one module that opens connections and runs statements (write-ahead logging, foreign keys on, one writer). A start-up check that the SQLite build has FTS5, with a clear message when it has not. |
| 4 | Commands `karazhan init`, `karazhan status` (sources, document and passage counts, schema version, size) and `karazhan doctor` (integrity check, orphan rows). Python API equivalents. |
| 5 | The workflow file `workflows/schema-changes.md`, its routing row, docs and release-notes entry. |

Sign-off: the tower location and override order, the four tables' columns and keys, the migration file
naming, the design envelope above, the output of `status` and `doctor`.

### Phase 3 — The Curator: ingestion

| Commit | Content |
|---|---|
| 1 | Source registry and the two v0.1 sources: a local directory and a git repository already cloned on disk (tracked files at the checked-out commit). `karazhan source add / list / remove`. |
| 2 | Parser registry and the v0.1 parsers: Markdown, plain text, reStructuredText, HTML (main text, headings kept), and docstrings and comments of Python files. Each returns a heading tree with line ranges. |
| 3 | Chunking: heading-aware passages with a size target and overlap, each carrying its heading path and line range; a passage never crosses a top-level heading. |
| 4 | `karazhan sync`: content-hash change detection, so an unchanged document is not re-parsed; changed documents replace their passages; documents gone from the source are removed. A summary of added, changed, removed and skipped. |
| 5 | Failure handling: an unreadable or oversized file is reported and skipped, never fatal; a `--dry-run` that writes nothing. |
| 6 | The workflow files `workflows/adding-a-source.md` and `workflows/adding-a-parser.md`, routing rows, docs and release-notes entry. |

Sign-off: the source and parser protocols, the file types and size limit, the chunk size target and overlap,
the passage identity (what makes a passage "the same" across syncs), the `sync` summary layout.

### Phase 4 — Attumen: lexical search

| Commit | Content |
|---|---|
| 1 | The FTS5 index over passages (text, heading path, document title), kept in step by `sync`; migration. |
| 2 | Query handling: plain words, quoted phrases, prefix; user input is escaped so that it can never break the FTS5 query syntax. BM25 ranking with column weights. |
| 3 | The result model shared by every later phase: passage text, score, document path, heading path, line range, source. Filters by source, path glob and file type. |
| 4 | `karazhan search "<query>"` with `--limit`, the filters, and `--json`; the Python API `Tower.search`. Human output shows the snippet with its citation. |
| 5 | Docs and release-notes entry. |

Sign-off: the tokenizer, the column weights, the result model's fields and the citation format (ask-first
from here on), the `search` flags and both output layouts.

### Phase 5 — Chess Event: evaluation

Built before the vector and hybrid phases on purpose: Attumen's scores become the baseline that Shade of Aran
and Medivh must beat.

| Commit | Content |
|---|---|
| 1 | The bundled evaluation corpus under `tests/`: a small, permissively licensed document set, and a golden set of questions with the passages that answer them. |
| 2 | Metrics: recall at k, mean reciprocal rank, nDCG at k, per question and aggregated; a retriever protocol so any search mode can be scored. |
| 3 | `karazhan eval` against the bundled set or a user's own golden file (documented format), with a table and `--json`. |
| 4 | The regression gate: the committed baseline scores, a test that fails when a metric drops below the baseline by more than the tolerance, and the step in `tools/verify.py`. |
| 5 | The workflow file `workflows/retrieval-changes.md`, its routing row, the pull request template's Retrieval scores section made binding, docs and release-notes entry. |

Sign-off: the corpus and its licence, the size of the golden set and who writes it, the metrics and the
values of k, the tolerance, the golden-file format.

### Phase 6 — Shade of Aran: embeddings and vector search

| Commit | Content |
|---|---|
| 1 | The embedder protocol and registry; a deterministic hashing embedder used by the test suite, so tests stay offline and reproducible. |
| 2 | The default local embedder as the optional extra `karazhan[embeddings]`; the model is downloaded only by an explicit `karazhan model pull`, never implicitly. |
| 3 | Vector storage in the tower (one vector per passage, tagged with the model's name and dimension) and `sync` embedding only new or changed passages; a model change marks vectors stale and `karazhan sync --reembed` rebuilds them. |
| 4 | Exact cosine search over the stored vectors; `karazhan search --mode vector`; the Python API. |
| 5 | Chess Event scores for the vector mode against the lexical baseline, recorded in the docs; docs and release-notes entry. |

Sign-off: the default model and runtime (*proposed:* a small ONNX sentence-embedding model run on the CPU,
no PyTorch dependency; the session checks current options and proposes one with its size and licence), the
vector storage format (*proposed:* float32 blobs and exact search with `numpy`; an approximate index is out
of v0.1), batch size, the behaviour when the extra is not installed (lexical search keeps working, with one
clear message).

### Phase 7 — Medivh: the query engine

The lead feature: one question in, the best cited passages out.

| Commit | Content |
|---|---|
| 1 | Hybrid retrieval: Attumen and Shade of Aran run on the same query and their rankings are fused (*proposed:* reciprocal rank fusion). Hybrid becomes the default mode when vectors exist; lexical is the fallback. |
| 2 | Result shaping: near-duplicate passages collapsed, adjacent passages of one document merged, a cap per document so one file cannot fill the list. |
| 3 | Context assembly: `karazhan ask "<question>"` returns the top passages within a size budget, each with its citation, in a human layout and as `--json` and `--markdown` (ready to paste into a prompt). The Python API `Tower.ask`. |
| 4 | An optional reranking step behind the same registry pattern, off by default. |
| 5 | Chess Event scores for hybrid against both single modes; the committed baseline moves to hybrid. Docs and release-notes entry. |

Sign-off: the fusion method and its constants, the duplicate and merge rules, the default budget and
per-document cap, the layouts of `ask`, whether reranking ships in v0.1 or moves to *Not in v0.1*, and the
minimum hybrid scores to ship.

### Phase 8 — Maiden of Virtue: hygiene

Runs before the MCP server on purpose: what an agent can reach is cleaned first.

| Commit | Content |
|---|---|
| 1 | Ignore rules: `.gitignore` honoured, a `.karazhanignore` file, built-in exclusions (binary files, dependency and build directories, `.env`, key and certificate files). |
| 2 | Secret redaction at ingestion: text matching well-known credential patterns is replaced before it is stored or embedded, and counted in the `sync` summary. |
| 3 | `karazhan purge` (remove a source, a path or everything matching a pattern from the tower, vectors included) and the documented guarantee that Karazhan never writes to a source, backed by a test. |
| 4 | Docs (the "what Karazhan stores and what it never does" section of the README and `SECURITY.md`) and release-notes entry. |

Sign-off: the built-in exclusion list, the redaction patterns and the replacement text, the behaviour on a
redaction match (redact and continue, *proposed*, or skip the file), the `purge` confirmation prompt.

### Phase 9 — Netherspite: the MCP server

| Commit | Content |
|---|---|
| 1 | `karazhan serve` over stdio as the optional extra `karazhan[mcp]`, on the official MCP Python SDK, read-only by construction (it opens the tower read-only). |
| 2 | Tools on the existing engine: search, ask, read a document section by citation, list sources. Results are returned as data with their citations and a size cap. |
| 3 | Tests driving the server through an in-process client; a documented set-up for Claude Code and other MCP clients. |
| 4 | The workflow file `workflows/adding-an-mcp-tool.md`, its routing row, hard rule 9 extended to the server in the checker, docs and release-notes entry. |

Sign-off: the tool names, descriptions and schemas (ask-first from here on), the result size cap, whether
`sync` is exposed as a tool (*proposed:* no, the server stays read-only).

### Phase 10 — Release v0.1.0

| Commit | Content |
|---|---|
| 1 | End-to-end test: init, add a source, sync, search, ask, eval, serve, on the bundled corpus. CI matrix extended to macOS and Windows and to each supported Python version. |
| 2 | README completed: quick start that works copy-paste, each codename's section, performance figures measured against the design envelope. |
| 3 | Packaging: build of the sdist and wheel checked in CI; a release workflow that publishes to PyPI on a version tag through trusted publishing (no stored token). |
| 4 | `RELEASE_NOTES.md`: the "in development" section becomes the v0.1.0 entry; the v0.1 section leaves this file; `SECURITY.md` supported versions set. |

Sign-off: the measured figures against the envelope, the PyPI project name and its trusted-publishing set-up
(done by the owner), the tag.

### Not in v0.1

- **Deferred, with a codename reserved:** Nightbane, answer synthesis by a language model, opt-in and summoned
  per call; Terestian Illhoof, crawling and fetching websites; Prince Malchezaar, a shared multi-user server
  with access control; Netherspite's HTTP portal; the Opera's second act, a web interface.
- **Deferred:** PDF and office documents; a watch mode that syncs on file change; an approximate vector index
  for towers beyond the design envelope; searching several towers at once; remote git cloning.
- **Out of scope:** a hosted service, telemetry of any kind, editing documents.

## After v0.1 — review before v0.2 is scoped

Starts when Phase 10 is merged and v0.1.0 is tagged; this section stays when the v0.1 section is removed.

- **Repository and codebase.** Every module against the hard rules, the module layout and the docs; dead
  code, duplicated helpers, stale docs and tests that no longer pin what they claim.
- **Retrieval.** The Chess Event scores re-run on a second, larger corpus the golden set was not written
  against, to check that the defaults were not tuned to the bundled one.
- **Guidance.** Each lesson recorded in the phases' pull requests against the guard it produced; prose that a
  checker now enforces is deleted and replaced by a pointer.

Read-only: findings are reported to the owner, ranked, with the evidence behind each; nothing is fixed
without sign-off. The v0.2 scope is agreed with the owner afterwards, starting from *Not in v0.1*.
