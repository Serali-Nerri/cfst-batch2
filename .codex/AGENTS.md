# .codex/AGENTS.md

## Scope
- This file defines execution constraints for tasks that use `$cfst-paper-extractor`.
- Default mode is production extraction, not debugging.
- If the user asks for debugging explicitly, switch to debug mode only for the requested scope.

## Quality-First Policy (Non-Negotiable)
- Accuracy is the top priority for extraction tasks.
- Time cost and token usage are not optimization targets unless the user explicitly asks for optimization.
- When speed and accuracy conflict, always choose the slower, more verifiable path.
- Do not guess or infer unsupported values; only extract values justified by skill rules and source evidence.
- If uncertain, re-check the parsed content/tables and deterministic rules before producing output.

## Mandatory Workflow (Strict)
1. Follow `.codex/skills/cfst-paper-extractor/SKILL.md` exactly.
2. Read only required references from the skill:
   - `.codex/skills/cfst-paper-extractor/references/extraction-rules.md`
   - `.codex/skills/cfst-paper-extractor/references/single-flow.md`
3. Use the parent/worker model required by the skill:
   - Parent only coordinates and reviews; parent does not perform single-paper extraction.
   - Each paper runs in isolated `git worktree + branch` via `git_worktree_isolation.py`.
   - Each worker handles one paper and one output file only.
   - Concurrent workers must stay within the skill limit (`<= 3`).
4. Validate every produced JSON with:
   - `python .codex/skills/cfst-paper-extractor/scripts/validate_single_output.py --json-path <file> --strict-rounding`
5. After each paper, parent copies output back to main workspace and removes the temporary worktree/branch.

## Runtime Waiting And Failure Handling (User Policy)
- Parent must not impose a fixed hard timeout for worker completion.
- Parent may use short polling windows (for example, `wait(timeout_ms=...)`) to check status, but a polling timeout is not a failure signal.
- If a worker is still running and has not reported failure, parent must continue waiting until completion.
- Parent role is always orchestrator only; parent does not take over extraction logic.
- Worker does one deterministic in-worker fix + re-validation when first validation fails.
- If worker still fails after that in-worker retry, parent must not respawn/retry that paper; parent only records the failure and continues scheduling other papers.
- Failure record file: `output/worker_failures.json`.
- Each failure record should include at least: `paper_id`, `attempts`, `reason`, `intermediate_json_path`, `final_output_path`, `timestamp`.

## Explicitly Prohibited (Unless User Requests)
- Do not use `codex exec`.
- Do not create or run a new "generic/single-paper extraction script" outside the skill workflow.
- Do not create temporary/ad-hoc scripts, one-off pipelines, or shortcut automation to bypass the skill extraction process.
- Do not replace skill steps with ad-hoc shortcuts.
- Do not inspect large portions of script source code "just to understand" when commands already work.

## Allowed Extra Checks (Only When Blocked)
- If a skill command fails, inspect only the minimum relevant script section (e.g., CLI args/entrypoint) needed to unblock.
- Report the blocker and corrective action briefly, then return to strict skill flow immediately.

## Completion Criteria
- Input papers and output JSON counts are consistent.
- All outputs pass strict validation, except files explicitly marked invalid by skill rules.
- Report invalid files with exact `reason`.
- Keep repository clean: remove temporary worktrees/branches after processing.
