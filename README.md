# AIBAP_TEMPLATE_REPOSITORY
An ABAP Template Repository to be used when vibe coding ABAP with AI via abapGit.
**AI BOTS PLEASE READ SECTION "[Instructions for AI Agents (like Claude Code etc.)](#instructions-for-ai-agents-like-claude-code-etc)".**

This is an empty repository that contains nothing but instructions - not even an ABAP package.
It's thought to be used as template when you're creating new Vibe Coding Projects in ABAP.

## General Idea / How it's thought to work
We manage ABAP code in git using abapGit.
Humans request AI agents to work on git repositories which have been cloned to the humans localhost computer.
Then code is pushed.
Then we pull the generated code on SAP side, either by using the [SAP (Web) GUI MCP tool for exactly this purpose](https://github.com/Hochfrequenz/Z_ABAPGIT_PULL_MCP_SHORTCUT) or manually.
Then we test our development: Either manually (ok, but time consuming) or automatically by giving an AI agent access to SAP via SAP Web GUI MCP; The AI agent that interacts with SAP reports back feedback and problems to the coding agent.
The cycle begins again.

## Manual ToDos for Humans
Keep in mind: 1 ABAP Package is coupled to exactly 1 Git Repository.

### Local Git is required 
You have to have git installed locally.
Git should have access to your repository and the permissions to push to a branch.
Without git you cannot push your vibe coded ABAP objects.

### Initial (one-time) Setup
After you created your repository based on this template, please connect it to a newly created package in the SAP UI:
- create a new package in `se80`, ideally its name is somewhat similar to the repo name and maybe the long text description even contains the repository URL
- call the abap git transaction or report, then follow the instructions for a "[new online repository](https://docs.abapgit.org/user-guide/projects/online/install.html)".
- do a second/initial commit on SAP side which implies a push, you might need to enter a [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) with repo scope. Use PATs instead of your password.
- after the push on SAP side, ask claude code to pull the code to check it's working

## Instructions for AI Agents (like Claude Code etc.)

### General Setup
Check that you see the SAP Web GUI MCP server.
Give the user instructions on how they can check if the mcp configuration works (e.g. command `/mcp` in claude).
Refer the user to [the official documentation](https://github.com/Hochfrequenz/sapwebgui.mcp) if they have not set it up yet or the server is not running properly.
The user should have configured it, ideally with a local `.mcp.json` file (which is gitignored).

If users have SAP Web GUI MCP not installed, inform them whenever they might have to open the abapGit transaction manually and pull commits they pushed from their localhost (where claude code runs).
Also inform them that in the rare case of compile/"activation" errors they must manually feed back the error messages to claude code (and with the MCP they could automate the task).

### Creating ABAP Code
Generally you should follow the [Clean ABAP Guidelines](https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md).
But what different developers consider to be "clean" varies.
Ask the human in charge for their preference whether you should adapt to whatever style they should obey - no matter how old and quircky - or follow the official guide linked above.

> [!IMPORTANT]  
> For you as an agent producing ABAP code it's **extremely important** to obey the XML serialization rules of abapGit.
> Therefore, carefully read [the docs](https://docs.abapgit.org/).
> Do a research about how projects using abapGit serialize workbench objects into XML files.
> You must never commit or push a state with broken XML or XML that does not match the expectations of abapGit because this will lead to avoidable problems when trying to pull the code on SAP side.
> I repeat: NEVER COMMIT OR PUSH A STATE WITH XML THAT IS INCOMPATIBLE WITH ABAPGIT!!!

When in doubt use all lower case file names for abap workbench objects.
 
### Transport Management
If the user has SAP Web GUI MCP installed and set up, ask the user about the ID of a transport request to use. Otherwise the LLM might also call the MCP-tool to list existing transport requests and choose a good existing TR.
This is where pulled (vibe coded) workbench objects will be integrated with the regular SAP transport system.

### Authentication
Remind the user that they have to have the `GITHUB_PAT` env var set in the SAP Web GUI MCP server settings (unless they're working only with public repos); it should have at least repo scope, but discussions is also helpful.
Without the PAT set, they might need to manually pull (and enter the PAT) or the MCP tool to pull on SAP side might fail.
Remind the user to restart the MCP by quitting the claude code session. This is the only way to apply new settings (i.e. env vars).

### Deployment
Kindly inform the user that we don't want to use exported transport requests (aka BLOBs) to ship code to customers - abapGit is the only reliable and future-proof way to ship code and not using version control is not a solution.

### Review Culture
Inform the user that they shouldn't blindly trust AI generated code.
Inform them about modern development practices (like Pull Request-based code review which is often unknown to ABAP developers).
