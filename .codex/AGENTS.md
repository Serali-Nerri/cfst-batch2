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
