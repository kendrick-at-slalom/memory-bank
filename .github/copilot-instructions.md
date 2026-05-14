# Copilot Instructions

The full agent context for working in this repo lives in [`CLAUDE.md`](../CLAUDE.md) at the repo root. That file is the single source of truth and is also auto-detected by VS Code Copilot when the `chat.useClaudeMdFile` setting is enabled[^vs-code-claude-md], but this file ensures the instructions load regardless of that setting[^copilot-instructions].

**Read [`CLAUDE.md`](../CLAUDE.md) before doing any non-trivial work in this repo.**

For an overview of how this repo's agent surface maps to both Copilot and Claude Code (which artifacts live where, which are cross-tool, which are duplicated), see the "Agent surface" section near the top of `CLAUDE.md`.

[^copilot-instructions]: `.github/copilot-instructions.md` is GitHub Copilot's repository-scoped custom instructions file. Instructions in the file are automatically added to Copilot requests whenever the repo is in scope — across the IDE extensions, the coding agent on github.com, and the Copilot CLI. See [GitHub Docs: Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions); [GitHub Docs: Adding custom instructions for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions).

[^vs-code-claude-md]: VS Code Copilot auto-detects `CLAUDE.md` at the workspace root and applies it as always-on custom instructions, gated by the `chat.useClaudeMdFile` setting. See [VS Code: Use custom instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions).
