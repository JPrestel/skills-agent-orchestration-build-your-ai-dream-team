# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/`, coordinated through GitHub Copilot CLI running in a Codespace.

- **Orchestrator** (`.github/agents/orchestrator.agent.md`) — Model: Claude Opus 4.7 (copilot). Coordinates the Planner, Coder, and Designer agents: breaks the request into phases, assigns non-overlapping file scopes, decides what can run in parallel vs. sequentially, and reports final results. Does not implement anything itself.
- **Planner** (`.github/agents/planner.agent.md`) — Model: Claude Opus 4.7 (copilot). Researches the repository and relevant docs, then produces an implementation plan: ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code.
- **Coder** (`.github/agents/coder.agent.md`) — Model: GPT-5.5 (copilot). Implements the application logic and structure within its assigned file scope, including support files like `.vscode/launch.json` for running the Project Pulse dashboard, and validates changes before reporting completion.
- **Designer** (`.github/agents/designer.agent.md`) — Model: Gemini 3.1 Pro (copilot). Owns UI/UX for the dashboard: information hierarchy, accessibility, responsive layout, and visual polish (project cards, status badges, priority treatment) using deterministic CSS hooks like `.dashboard` and `.project-card`.

All four agents are restricted from staging, committing, or pushing changes — I control git operations directly through Copilot CLI prompts.
