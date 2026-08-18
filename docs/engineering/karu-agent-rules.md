# Karu Agent Rules

Canonical cross-project operating rules for AI agents working on Karu repositories.

These rules are project-agnostic. Individual repositories may add stricter project-specific rules in their own `AGENTS.md`, but should not silently weaken these defaults.

## 1. Reliability and resumable execution

Long tool-heavy turns are a reliability risk. Optimize for resumable work instead of maximizing the amount completed in one uninterrupted response.

### Work in bounded batches

- Break large investigations or repository changes into small checkpoints with a clear completed state.
- Prefer at most **2–3 repository mutations** in one batch before verifying remote state.
- After a mutation batch, re-read the affected PR, branch, file, commit, workflow, or issue state before continuing.
- Do not create an unbounded chain of search/fetch/write calls merely to finish everything in one turn.
- Prefer targeted file/range reads over broad repository dumps or repeated full-diff retrievals.

### Treat interrupted writes as potentially successful

If a tool response, stream, connector call, or assistant response fails after a write may have been sent:

1. **Do not immediately repeat the write.**
2. Read the authoritative remote state first.
3. If the intended mutation already exists, continue from that state without duplicating it.
4. Retry only the missing operation.

This applies to commits, branch updates, PR edits, merges, comments, labels, file writes, CI reruns, releases, deployments, and other side-effecting operations.

### Stop at a safe checkpoint

If the remaining work would require a substantially larger tool burst than the completed batch, leave the project in a verified checkpoint and report exactly what is complete and what remains.

A clean resumable checkpoint is preferred over a fragile all-at-once run.

Never claim a write, merge, CI result, deployment, or remote state succeeded unless it has been verified after the operation.

## 2. Git and GitHub safety

- Inspect the current base branch and relevant callers/tests before changing an existing interface.
- Re-read files after concurrent work lands; never force an older snapshot over newer work.
- Avoid history rewrites unless explicitly required and understood.
- Prefer additive, reviewable commits over broad rewrites.
- Keep PR scope coherent. Unrelated changes should be split.
- Before retrying a failed GitHub operation, verify whether GitHub already accepted it.
- Prefer merging or rebasing current `main` deliberately before continuing work on a branch known to be stale or conflict-prone.
- Do not delete files or branches merely to simplify conflict resolution unless deletion is part of the intended change.

## 3. Source, documentation, and evidence separation

Keep independently moving workstreams separate whenever practical.

Recommended separation:

- production/source code and tests;
- reverse-engineering or research evidence;
- generated artifacts;
- operational/configuration changes;
- documentation and status/changelog updates.

Avoid making frequently edited status files a required participant in every source-code PR. Hotspot documents should be updated in a focused documentation change when possible.

A documentation conflict must not be resolved by overwriting newer source facts with an older branch snapshot.

## 4. Source of truth

For project-specific behavior, follow the repository's declared evidence hierarchy.

General defaults:

1. Direct authoritative evidence for the project.
2. Direct observed behavior.
3. Official specifications or primary-source documentation.
4. Trusted implementations and secondary references.
5. Reasoned inference.

Do not silently promote inference into confirmed behavior.

When correcting an earlier conclusion, update the canonical source instead of leaving contradictory conclusions scattered across the repository.

## 5. Efficient context and tool use

- Read only what is needed for the current decision.
- Reuse already retrieved identifiers, SHAs, PR numbers, paths, and remote state instead of rediscovering them.
- Prefer one targeted query over several speculative queries.
- Avoid repeatedly fetching unchanged large files or diffs.
- Do not run CI, builds, or workflows merely for a documentation-only change unless required by repository policy.
- When a cheap local/static verification can establish the needed fact, prefer it over an expensive workflow run.

## 6. Expensive CI and GitHub Actions

GitHub Actions minutes are a shared resource.

- Avoid triggering Actions for trivial metadata or documentation-only edits unless necessary.
- Batch related source changes before pushing when that reduces redundant workflow runs without making the change harder to review.
- Prefer re-running only failed jobs when supported rather than re-running a full successful workflow.
- Do not repeatedly amend/push a branch solely to observe CI behavior when logs or local reproduction can answer the question.
- Verify workflow scope and path filters before introducing new automatic runs.

## 7. Project-local overrides

Each repository may maintain its own `AGENTS.md` for domain-specific requirements.

Repository-local rules should contain only what is genuinely specific to that project, for example:

- architecture constraints;
- product invariants;
- platform restrictions;
- domain-specific safety requirements;
- evidence hierarchy exceptions;
- build/test commands;
- file ownership or generated-file rules.

Cross-project reliability and GitHub operating rules should remain canonical here rather than being independently rewritten in every repository.

## 8. Rule synchronization

`wiki-karu-web/docs/engineering/karu-agent-rules.md` is the canonical human-readable copy of Karu-wide agent rules.

Because `AGENTS.md` does not automatically inherit rules across GitHub repositories, repositories that need machine-local enforcement should carry a synchronized Karu-global section or generated local copy plus their project-specific rules.

When the global rules change:

1. Update this canonical document first.
2. Sync only the global section into participating repositories.
3. Preserve each repository's project-specific rules.
4. Verify the resulting diff before committing.

Do not maintain divergent hand-edited copies of the same global policy when a synchronized copy can be used.
