# CLAUDE.md

Project-specific guidance for AI agents working in a repository created from this template. Read this alongside [README.md](README.md).

This file is part of the template and gets copied into every new project. Adjust it to your project's real constraints (package name, system, style) after you create the repo.

## What this repo is

A git repository that holds the ABAP source of **one** ABAP package. Code is written by AI agents (you), reviewed by humans in pull requests, and synchronized with a SAP system either:

- **via ADT** — you edit SAP directly through [`aibap.mcp`](https://github.com/Hochfrequenz/aibap.mcp) and pull the resulting state back into git for review, or
- **via abapGit** — you write files in this working tree, push to git, and abapGit pulls them into SAP.

Both workflows are supported. The README describes when each applies.

## Pick the right MCP for the job

Before you start editing, check which MCPs are actually available in this session (`/mcp` in Claude Code). You may have one, the other, or both.

| Task | Prefer |
|---|---|
| Read, write, activate, syntax-check, test ABAP code | `aibap.mcp` (ADT) |
| Create a new object (`PROG`, `CLAS`, `INTF`, `FUGR`, `MSAG` — all systems; `DDLS`, `TABL`, `DTEL`, `DOMA` — S/4HANA only) | `aibap.mcp` (`create_object`) |
| Transport management (create, assign, release) | `aibap.mcp` (`transport` group) |
| ATC / ABAP Unit / syntax check | `aibap.mcp` |
| Short dumps / ST22 analysis | `aibap.mcp` (`shortdumps` group) |
| Pulling a git branch into SAP via abapGit | `sapgui.mcp` (`sap_abapgit_pull`) — requires the `Z_ABAPGIT_PULL_MCP_SHORTCUT` report on SAP side |
| Running an arbitrary transaction, looking at a screen | `sapgui.mcp` (`sap_transaction`, browser tools) |
| Customizing / table maintenance screens | `sapgui.mcp` |

If a tool you need is not visible, tell the user — they may need to enable a tool group (e.g. `--tools=...` for `aibap.mcp`) or start the other MCP.

## Rules of the road

### Keep package documentation current
Every PR that adds or changes an extension point or API endpoint must update `docs/extension-points.md` and/or `docs/api.md` in the same PR.
Reviewers check that these references match the code.

### Do not hand-write abapGit XML
If you are tempted to write a `*.clas.xml`, `*.prog.xml`, etc. from scratch: stop. Either

1. use `aibap.mcp`'s `create_object` to let SAP serialize the object and then pull the result into git via abapGit, or
2. copy the structure from an existing object of the same type in the same repo.

Broken abapGit XML will cause pull failures on SAP side. Never commit a state whose XML you are not sure abapGit will accept.

### File naming
Use **all lowercase** file names for ABAP workbench objects (e.g. `zcl_my_service.clas.abap`, not `ZCL_MY_SERVICE.clas.abap`). abapGit is case-sensitive on Linux/macOS checkouts.

### Clean ABAP style
Default to [Clean ABAP](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md), but ask the human about the house style first — see README → "Creating ABAP Code" for the rationale.

### Keep one original language for all objects
Every workbench object has an original language (TADIR field `MASTERLANG`), and its texts (descriptions, DDIC texts, program titles) are stored per language.
A new object gets the logon language of the session that creates it, so an agent logged on in English creates English objects in a German package without noticing.
Mixed languages cause trouble later: abapGit writes the language into the XML (`LANGU`, `DDLANGUAGE`, `MASTERLANG`), and SAP asks "logon language differs from original language" for every such object when you pull or edit it in the other language.

- Agree on the package's original language with the user once, before creating the first object, and set it as `MASTER_LANGUAGE` in `.abapgit.xml` (the one-letter SAP language key, e.g. `E` or `D`).
- Make every session that creates objects log on in that language, both the ADT MCP and the SAP GUI MCP (the `language` setting of the system in the MCP configuration).
  For ADT, check the result rather than trusting the setting: the logon language may be taken from the HTTP `Accept-Language` header, not only from `sap-language`.
- After creating an object, check its original language (TADIR `MASTERLANG`) and that its description exists in that language.
- Check the serialized XML before committing: all language tags must carry the agreed language.
  The [original-language workflow](.github/workflows/language-check.yml) checks XML under `src/` on every pull request.
  Empty paired language tags fail, while self-closing language tags pass.
  Language tags nested under elements whose names start with `I18N_` are skipped.
  Set the repository variable `ABAP_CHECK_I18N_LANGUAGES` to `true` to check them as well, because translations legitimately carry other languages.
  The workflow uses `MASTER_LANGUAGE` from `.abapgit.xml` when present.
  Otherwise, set the `ABAP_ORIGINAL_LANGUAGE` repository variable (default `E`).
- If an object already has the wrong original language, tell the user before changing it (for example via SE03 "change object directory entry" where the release allows it, otherwise with Basis support).
  The texts do not move with the language: after the change, the descriptions and DDIC texts must be maintained again in the new language.

### Never commit secrets
`.mcp.json`, `opencode.json`, `systems.json`, PATs, passwords — none of these belong in the repo. The template's `.gitignore` already excludes `.mcp.json`, `opencode.json`, and Claude Code local settings.

### Respect the transport system
Every change lands in a transport request — ask the user which TR to target, and **never release a transport without explicit permission**. See README → "Transport Management" for the per-workflow details.

### Write reviewable commits
Keep commits reviewable: one logical change per commit, descriptive message, and a PR description that explains *why*. See README → "Review Culture" for why PR review matters in this project.

### Stacked PRs and shared dev system
When several PRs use the same dev system in Workflow A, a whole-package export or sync can bring in-flight objects from other PRs into your branch.

- Commit only files belonging to your task and stage them explicitly with `git add <paths>` — never use `git add -A` after a package sync.
- Before committing, compare all changed files (including untracked files) with the list of objects you touched in this task. Drop unrelated sync output or stash it if it must be preserved; do not discard someone else's local work.
- If the sync tooling supports an object or file list, pass that list to the export instead of exporting the whole package.

Related MCP work: [Hochfrequenz/aibap.mcp#534 — better abapGit support](https://github.com/Hochfrequenz/aibap.mcp/issues/534).

## When things go wrong

- **Syntax error on activate:** fix it before moving on. Don't commit code that doesn't activate.
- **423 lock errors via ADT:** the object is locked by someone else, or SAP is checking the lock against the wrong enqueue table. `aibap.mcp` auto-locks; if a persistent 423 remains on a fresh object you own, the fix is usually a stateful session (`X-sap-adt-sessiontype: stateful`) — check `aibap.mcp`'s own CLAUDE.md for current guidance.
- **abapGit pull fails:** most likely XML serialization issue (see "Do not hand-write abapGit XML" above) or a missing PAT. Read the error from `sap_abapgit_pull` or the status bar before guessing.
- **Popup "logon language differs from original language" on every pull:** objects were created in another language; see "Keep one original language for all objects" above.
- **MCP server not visible:** the user may need to restart Claude Code after editing `.mcp.json` or env vars.

## Project-specific notes

<!-- Fill these in for your concrete project: -->

- **Package:** `<ZYOUR_PACKAGE>`
- **System(s):** `<dev>`, `<qa>`, `<prod>`
- **Style:** Clean ABAP / house style / mixed
- **Test coverage expectations:** e.g. "all classes in `src/` must have an ABAP Unit test"
- **Transport discipline:** e.g. "one TR per PR", "share TR `ABCK000123`"
