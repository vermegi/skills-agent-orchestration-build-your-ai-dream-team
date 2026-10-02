# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/`, orchestrated with the GitHub Copilot CLI running in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases, assigns non-overlapping file scopes, runs independent tasks in parallel and dependent/overlapping tasks sequentially, verifies the integrated result, and reports progress and blockers. Does not implement work itself and never stages/commits/pushes changes.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository, docs, dependencies, and edge cases, then produces an implementation plan with ordered steps, file assignments, dependencies, parallel vs. sequential work, risks, and open questions. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code-oriented tasks within the file scope assigned by the Orchestrator, including support configuration like `.vscode/launch.json` for Project Pulse (pointing `cwd` to `${workspaceFolder}/app` and opening `index.html`). Keeps changes explicit, deterministic, and validated before reporting completion.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design within its assigned scope. For Project Pulse, builds a polished dashboard with project cards, status badges, priority treatment, and deterministic CSS hooks such as `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`

## Orchestration note

All four agents are run and coordinated using the GitHub Copilot CLI inside a Codespace — the Orchestrator delegates to Planner, Coder, and Designer, while the learner retains full control over git operations (staging, committing, and pushing).
