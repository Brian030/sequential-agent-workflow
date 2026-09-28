---
name: sequential-agent-workflow
description: Run a user-defined Markdown workflow in one Codex task, assigning each stage to a fresh subagent with its specified model and reasoning effort, verifying acceptance criteria, and stopping on failure. Use when the user asks to run or reuse a staged subagent workflow.
---

# Sequential agent workflow

Run the stages in a Markdown workflow file inside the current Codex task. The main agent coordinates and verifies; one fresh subagent performs each stage. [workflow.md](workflow.md) is the editable default workflow and an example of the expected format.

## Find and check the workflow

- Use the workflow path supplied by the user. If none is supplied, use the `workflow.md` alongside this skill. Do not substitute another Markdown file without the user's direction.
- Treat the selected file as the source of truth. It must contain one or more `## 任務 N：名稱` sections in execution order. Every section must have a model, reasoning effort, nonempty `### 任務內容`, and nonempty `### 完成標準`. Acceptance criteria must be specific enough to check against observable output. Optional `### 交付成果` can name expected files or results. Reject unedited template placeholders such as `請填寫` or `請描述`.
- Before starting any subagent, check all stages for missing or conflicting fields, duplicate numbers, unavailable models, and unsupported model/effort combinations using the capabilities of the current Codex environment. On an invalid workflow, stop and report the exact stage and field. Do not silently choose a different model, effort, task, or standard.
- The workflow may contain any positive number of stages. Do not invent additional stages or change their order. Use the workflow as it stood when the run began unless the user explicitly revises it.

## Execute and gate each stage

1. Start only the current stage's subagent. Use the stage's requested `model` and `reasoning_effort`, and `fork_turns: "none"` so it does not inherit the main agent's accumulated conversation. Include the stage task, standards, needed constraints, and paths to relevant accepted artifacts in its assignment. Do not forward entire prior subagent transcripts. Instruct it not to delegate or begin another stage.
2. Wait for the subagent's actual final result. A successful creation or wait call alone is not proof of completion. Request a concise result naming deliverables, work performed, evidence, and blockers.
3. The main agent checks every completion criterion independently. Run specified tests or inspect specified artifacts when applicable. A subagent's statement that it succeeded is not itself evidence. Record a brief pass/fail result and evidence for each criterion.
4. Only after all criteria pass, start the next stage. If any criterion fails, cannot be verified, the subagent fails, or a required tool is unavailable, stop immediately. Do not retry or start another subagent unless the user asks. Tell the user in this Codex task which stage stopped, why, what evidence was checked, and which outputs remain available.
5. After the final stage passes, report the completed stages and final deliverables.

Keep detailed artifacts in the workspace and pass later subagents only the files and facts they need. The coordinator may retain a concise stage ledger, but should not paste full prior histories into new assignments. Do not create separate Codex sidebar tasks for these stages.
