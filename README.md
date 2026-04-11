# AIBAP_TEMPLATE_REPOSITORY
An ABAP Template Repository to be used when vibe coding ABAP with AI.
**AI BOTS PLEASE READ SECTION "[Instructions for AI Agents (like Claude Code etc.)](#instructions-for-ai-agents-like-claude-code-etc)".**

This is an empty repository that contains nothing but instructions — not even an ABAP package.
It is meant to be used as a GitHub template when you create a new vibe coding project in ABAP.

## General Idea / How it's thought to work
We manage ABAP code in git, and we let AI agents do the actual writing.
There are two complementary workflows an agent can use, and this template supports both — pick whichever matches what your SAP system and your local MCP setup support.

### Workflow A — ADT (via [`mcp-server-abap`](https://github.com/Hochfrequenz/mcp-server-abap))
The agent talks to the **SAP ADT (ABAP Development Tools) REST API** directly.
It can read and write source, run syntax checks, activate objects, run ABAP Unit tests, run ATC, manage transports, and look at runtime errors — all without touching SAP GUI.
The loop is tight: edit → syntax check → activate → test — all via MCP tool calls.
Git still matters for code review and history: you commit the pulled-back state to the repo so humans can review it in a PR.

**Requires:** the SAP system must have the ADT services active (transaction `SICF`: `/sap/bc/adt/*`), the user must have developer authorizations, and [`mcp-server-abap`](https://github.com/Hochfrequenz/mcp-server-abap) must be configured in your MCP client.

### Workflow B — abapGit roundtrip (via [`sapwebgui.mcp`](https://github.com/Hochfrequenz/sapwebgui.mcp))
The agent writes ABAP files locally, commits and pushes them to git, and then the changes are pulled into SAP via abapGit.
Pulling to SAP can be done with the [SAP (Web) GUI MCP tool for exactly this purpose](https://github.com/Hochfrequenz/Z_ABAPGIT_PULL_MCP_SHORTCUT) (which requires the `Z_ABAPGIT_PULL_MCP_SHORTCUT` report installed on SAP side), or manually via the abapGit transaction.
Testing and feedback happen through [`sapwebgui.mcp`](https://github.com/Hochfrequenz/sapwebgui.mcp) — the agent can navigate SAP, read the status bar, and report back problems.
The cycle then repeats.

**Requires:** an abapGit-capable SAP system (abapGit installed), a registered online repository, and [`sapwebgui.mcp`](https://github.com/Hochfrequenz/sapwebgui.mcp) configured — optionally with the [`Z_ABAPGIT_PULL_MCP_SHORTCUT`](https://github.com/Hochfrequenz/Z_ABAPGIT_PULL_MCP_SHORTCUT) report installed for MCP-driven pulls.

### Which one should I use?
Both are valid — use whichever matches your system and your tooling.

| | Workflow A (ADT) | Workflow B (abapGit) |
| --- | --- | --- |
| **Source of truth between commits** | SAP | local git |
| **Write to SAP** | `mcp-server-abap` tools (`set_source_from_file`, `patch_source`, `create_object`) | `git push` → abapGit pull |
| **Activate / test** | `mcp-server-abap` (`activate_object`, `syntax_check`, `run_unit_tests`, `run_atc_check`) | via `sapwebgui.mcp` or manually |
| **Transport management** | `mcp-server-abap` `transport` tool group | manually (or via `sapwebgui.mcp`) |
| **XML correctness risk** | none — SAP owns serialization | high — the agent must write abapGit-compatible XML |
| **Needs on SAP side** | ADT services active | abapGit installed + repo registered |

The two workflows can also be combined.
A common pattern: use **Workflow A** for the fast edit/activate/test loop during development, and use abapGit (**Workflow B**) to pull the state back into the git repo so humans can review it in a pull request.

> [!TIP]
> **Hochfrequenz colleagues:** internal setup documentation for both MCPs is at <https://brain.hochfrequenz.de/books/ki-tools-bei-hochfrequenz/chapter/sap-mcps>.

## Manual ToDos for Humans
Keep in mind: 1 ABAP package is coupled to exactly 1 git repository.

### Local git is required
You have to have git installed locally.
Git should have access to your repository and the permissions to push to a branch.
Without git you cannot push your vibe coded ABAP objects (and even in Workflow A you will want git for code review).

### Initial (one-time) setup
After you created your repository based on this template, connect it to an ABAP package on SAP side.

#### Register the repository on SAP side (Workflow B)
If you plan to use the abapGit workflow, you have to register this repository as an **online repository** in abapGit.
The official abapGit documentation is well maintained — follow it instead of duplicating steps here.

- Create a new package in `se80`.
- Call the abapGit transaction or report and follow the instructions for registering a [new online repository](https://docs.abapgit.org/user-guide/projects/online/install.html).
- Do an initial commit on SAP side (this implies a push).
- For private repositories you will need a [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) with `repo` scope — use PATs instead of your password.
- After the push on SAP side, ask your AI agent to pull the code to verify everything works.

#### Create the package on SAP side (Workflow A)
If you plan to use only the ADT workflow, you still need a package to create objects in.
Create one in `se80` (or via whatever mechanism your team uses) — `mcp-server-abap` does not currently expose a "create package" or "register abapGit repo" tool.

## Instructions for AI Agents (like Claude Code etc.)

### General Setup
Check which SAP MCPs are available in the current session.

- [`mcp-server-abap`](https://github.com/Hochfrequenz/mcp-server-abap) — ADT REST API, for direct code operations (read, write, activate, test, transports).
- [`sapwebgui.mcp`](https://github.com/Hochfrequenz/sapwebgui.mcp) — SAP (Web) GUI automation, for things the ADT API cannot do (running arbitrary transactions, abapGit pull, customizing screens).

Give the user instructions on how they can check if the MCP configuration works (e.g. command `/mcp` in Claude Code).
Refer the user to the official documentation of the respective MCP if they have not set it up yet or the server is not running properly.
The user should have configured their MCPs locally, ideally with a project-local `.mcp.json` file (which is gitignored by this template).

**Hochfrequenz colleagues:** internal setup documentation for both MCPs is at <https://brain.hochfrequenz.de/books/ki-tools-bei-hochfrequenz/chapter/sap-mcps>.

### Creating ABAP Code
Generally you should follow the [Clean ABAP Guidelines](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md).
But what different developers consider to be "clean" varies.
Ask the human in charge for their preference whether you should adapt to whatever style they should obey — no matter how old and quirky — or follow the official guide linked above.

#### If you are using Workflow A (ADT via `mcp-server-abap`)
Prefer ADT tools to edit ABAP objects on SAP directly.
SAP handles all serialization for you, so you do not have to worry about abapGit XML at all.

- Create new objects with `create_object`.
- Supported types on all systems: `PROG`, `CLAS`, `INTF`, `FUGR`, `MSAG`.
- Additionally on S/4HANA only: `DDLS`, `TABL`, `DTEL`, `DOMA` — these will not work on ECC.
- Read source with `get_source`, `get_class_definition`, or `get_include_source`.
- Write source with `set_source_from_file`, `set_include_source`, or `patch_source` (all auto-lock).
- Format with `pretty_print`.
- Syntax-check with `syntax_check` and `verify_source` before activating.
- Activate with `activate_object` / `activate_objects`.
- Run tests with `run_unit_tests` and static analysis with `run_atc_check`.
- Manage transports with the `transport` tool group (`get_transport_requests`, `create_transport`, `add_to_transport`, `release_transport`, etc.).

Tool availability depends on which tool groups the user has enabled in `mcp-server-abap`.
If a group you need (e.g. `debug`) is not loaded, tell the user so they can enable it.

#### If you are using Workflow B (abapGit roundtrip)

> [!IMPORTANT]
> For you as an agent producing abapGit-serialized ABAP files, it is **extremely important** to obey the XML serialization rules of abapGit.
> Carefully read [the docs](https://docs.abapgit.org/).
> Research how projects using abapGit serialize workbench objects into XML files.
> You must never commit or push a state with broken XML or XML that does not match the expectations of abapGit — this will lead to avoidable problems when trying to pull the code on SAP side.
> I repeat: NEVER COMMIT OR PUSH A STATE WITH XML THAT IS INCOMPATIBLE WITH ABAPGIT!!!

When in doubt use all lower case file names for ABAP workbench objects.

**Tip:** if you also have access to `mcp-server-abap`, a reliable way to avoid hand-writing abapGit XML is to create the object on SAP first via `create_object`, then pull it back into the git repo via abapGit.
The SAP side does the serialization correctly for you.

### Pulling Code to SAP (Workflow B)

> [!NOTE]
> This section applies if the user has [`sapwebgui.mcp`](https://github.com/Hochfrequenz/sapwebgui.mcp) installed (WebGUI or Desktop backend) **and** the [`Z_ABAPGIT_PULL_MCP_SHORTCUT`](https://github.com/Hochfrequenz/Z_ABAPGIT_PULL_MCP_SHORTCUT) report installed on SAP side.
> If you do not see the tools mentioned below, the user may be using a different MCP server, might not have the shortcut report installed, or uses no MCP at all — in that case, ask them to pull manually via the abapGit transaction.

After pushing ABAP code to the git repository, pull it to SAP using these MCP tools:

1. **Discover the repo name:** Call `sap_abapgit_list_repos()` to see all registered abapGit repositories with their names, Git URLs, and packages.
2. **Pull:** Call `sap_abapgit_pull(repo="<REPO_NAME>")` to pull changes from git to SAP. The `repo` parameter is pattern-matched against registered repository names, so a substring is sufficient.
   - If SAP requires a transport request, the tool returns an error with guidance. Retry with `sap_abapgit_pull(repo="<REPO_NAME>", trkorr="<TRANSPORT_ID>")`.
   - For private repos, ensure `GITHUB_PAT` or `ABAPGIT_PAT` is set in the MCP server's environment (see Authentication below).
3. **Verify:** After a successful pull, use `sap_read_status_bar()` to confirm the result, or navigate to the pulled objects (e.g., `sap_transaction("SE38")`) to check the code.

These tools work on both the WebGUI and Desktop backends of `sapwebgui.mcp`.

### Transport Management
- **Workflow A:** use the `transport` tool group in `mcp-server-abap` to list, create, assign to, and release transport requests.
  Ask the user which transport to use, or call the list tool to pick a sensible existing one.
- **Workflow B:** ask the user about the ID of a transport request to use.
  If the user has `sapwebgui.mcp` set up, you can also navigate to `SE09`/`SE10` via `sap_transaction` to review transports.
  Pulled (vibe-coded) workbench objects will be integrated with the regular SAP transport system when they land in the TR.

### Authentication
**For `sapwebgui.mcp` abapGit pulls:** remind the user that they have to have the `GITHUB_PAT` or `ABAPGIT_PAT` env var set in the MCP server settings (unless they are working only with public repos).
It should have at least `repo` scope; `discussions` is also helpful.
Without the PAT set, they might need to manually pull (and enter the PAT), or the MCP tool to pull on SAP side might fail.
Remind the user to restart the MCP by quitting the Claude Code session.
This is the only way to apply new settings (i.e. env vars).

**For `mcp-server-abap`:** credentials are read from `~/.config/sap-mcp/systems.json` (see [sap-mcp-config](https://github.com/Hochfrequenz/sap-mcp-config) for the file format, which is shared with `sapwebgui.mcp`).
Both user/password and OAuth2 (SAML SSO) are supported — refer the user to the `mcp-server-abap` README if they need to set up OAuth2.

### Deployment
Kindly inform the user that we don't want to use exported transport requests (aka BLOBs) to ship code to customers — abapGit is the only reliable and future-proof way to ship code, and not using version control is not a solution.

### Review Culture
Inform the user that they shouldn't blindly trust AI generated code.
Inform them about modern development practices (like pull-request-based code review, which is often unknown to ABAP developers).
Even in Workflow A, where SAP is the live edit target, the abapGit pull into git + PR review is the mechanism by which humans keep control of what the AI actually did.
