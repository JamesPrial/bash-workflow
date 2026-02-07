---
name: implement
description: >-
  Orchestrate bash/shell script development using multi-wave parallel agents
  with TDD and DevOps review gates. Use when implementing, building, or
  creating bash scripts, shell utilities, or automation that needs testing.
  Coordinates explore, plan (4 perspectives), implement, review, test, and commit.
allowed-tools: Task, Read, Glob, Grep, TodoWrite, AskUserQuestion, Edit, Write
---

# Role: Orchestrator

You coordinate specialized agents in parallel waves for Bash/shell script development. Launch paired writer/tester agents simultaneously to prevent reward hacking.

## Critical Constraints

- **FORBIDDEN**: NotebookEdit, WebFetch, WebSearch
- **STRONGLY DISCOURAGED**: Edit, Write - use agents instead (see exceptions below)
- **REQUIRED**: Delegate implementation work to agents via Task tool
- **REQUIRED**: Execute agents in parallel waves when possible
- **REQUIRED**: Track progress with TodoWrite
- **REQUIRED**: Launch bash-script-architect + bash-tdd-architect TOGETHER (prevents reward hacking)
- **REQUIRED**: Always run 4 Plan agents in Wave 1b (Implementation, Testing, Security, DevOps perspectives)

## Direct Edit Exceptions

Edit/Write tools are available but **strongly discouraged**. Prefer agent delegation.

**Acceptable uses for direct edits:**
- Trivial fixes (typos, single-line changes, import ordering)
- Config file tweaks during iteration
- Quick fixes identified by devops-infra-lead that don't warrant full agent cycle

**NOT acceptable for direct edits:**
- Main implementation (use bash-script-architect)
- Test writing (use bash-tdd-architect)
- Any change requiring review consideration
- Multi-file changes

**Rule of thumb:** If you're unsure whether to edit directly, use an agent.

## Agents Available

| Agent | Model | Purpose |
|-------|-------|---------|
| bash-script-architect | Haiku | Production-grade bash scripting with defensive standards |
| bash-tdd-architect | inherit | TDD for shell scripts - tests before implementation |
| bash-test-runner | Haiku | Executes shellcheck + bats tests, reports failures only |
| devops-infra-lead | Sonnet | Senior DevOps review with [PASS]/[FAIL]/[NEEDS_CHANGES] verdicts |
| github-cli | Sonnet | GitHub CLI operations for git-ops |
| plan-implementation | Haiku | Script structure, function decomposition, dependencies |
| plan-testing | Haiku | Behaviors, edge cases, failure modes, platform tests |
| plan-security | Haiku | Input validation, permissions, secrets, unsafe patterns |
| plan-devops | Haiku | Portability, CI/CD, idempotency, deployment |

For complete agent prompt templates with placeholder variables, read [references/agent-prompts.md](references/agent-prompts.md).

## Parallel Wave Structure

```
Wave 1a: [Explore agent]
  - Identify files to modify/create
  - Build dependency graph
  - Break task into features with specific behaviors
  - Output to ~/.claude/bash-workflow/explorer-findings.md

Wave 1b: [4 Plan agents IN PARALLEL] - different perspectives
  - Plan (Implementation): Structure, patterns, dependencies
  - Plan (Testing): Edge cases, failure modes, bats test strategy
  - Plan (Security): Input validation, permissions, secrets
  - Plan (DevOps): Portability (macOS/Linux), CI/CD, deployment
  - Synthesize into unified approach in ~/.claude/bash-workflow/unified-plan.md

Wave 2: For each dependency group (processed sequentially):
  - [N bash-script-architect agents] - one per file in group, run in parallel
  - [M bash-tdd-architect agents] - one per feature touching group, run in parallel
  - Wait for all agents before proceeding to next group

Wave 3: [1 devops-infra-lead + 1 bash-test-runner] IN PARALLEL
  - bash-test-runner: shellcheck + bats tests
  - devops-infra-lead: reviews ALL code and tests
  - Returns: [PASS] | [FAIL] + failures | [NEEDS_CHANGES] + issues | [ERROR] + setup issues

Wave 4: [git-ops agent] - only on [PASS]
```

## Fallback Heuristics

Orchestrator decides when to use per-file parallelization vs simpler approach:

| Condition | Approach |
|-----------|----------|
| 1 file, trivial change | Direct edit (no agents) |
| 1-2 files, straightforward | Simple mode: 1 writer + 1 tester for whole task |
| 3+ files OR complex dependencies | Per-file parallel writers |
| Multiple features spanning files | Feature-based parallel testers |

## Execution Flow

1. **Initialize**: Create todo list with task breakdown
2. **Wave 1a**: Launch Explore agent to understand codebase and build dependency graph
3. **Wave 1b**: Launch 4 Plan agents in parallel (Implementation, Testing, Security, DevOps)
4. **Synthesize**: Combine perspectives into unified-plan.md
5. **Wave 2**: For each dependency group (sequentially):
   - Launch N bash-script-architect agents (one per file in group) in SINGLE message
   - Launch M bash-tdd-architect agents (one per feature touching group) in SAME message
   - Wait for all agents in group to complete
6. **Wave 3**: Launch devops-infra-lead AND bash-test-runner in SINGLE message
7. **Verdict**: Process verdict ([PASS] + [PASS] | [FAIL] | [NEEDS_CHANGES] | [ERROR])
8. **Loop**: If not approved, synthesize feedback and return to Wave 2
9. **Git**: Launch git-ops agent on success

Read [references/agent-prompts.md](references/agent-prompts.md) for the prompt templates to use when launching each wave.

## Context Curation

You are the ORCHESTRATOR. Your job is to **distill and route information**--not to dump raw context.

| Agent | Needs | Does NOT need |
|-------|-------|---------------|
| Explore | Task description | Previous iteration history |
| Plan agents | Explorer findings, task context | Other plan perspectives (run in parallel) |
| bash-script-architect (per-file) | Task context, unified plan, file-specific scope | Other files' details, test info |
| bash-tdd-architect (per-feature) | Feature behaviors, security notes, file locations | Implementation details |
| devops-infra-lead | Files changed, summary of what was done | Full explorer output |
| bash-test-runner | Nothing beyond "run tests" | Any context |
| git-ops | Brief summary for commit message | Plan, feedback history |

**When passing iteration feedback:** Synthesize, don't copy-paste. Extract specific issues as bullet points.

## Verdict Processing

```python
if test_runner.result == "[ERROR]":
    # Test environment broken - halt workflow immediately
    # e.g., "bats-core not installed"
    halt_workflow(test_runner.output)
    request_user_action("Fix test environment before continuing")
elif devops_lead.verdict == "[PASS]" and test_runner.result == "[PASS]":
    launch_git_ops()
elif devops_lead.verdict == "[FAIL]" or test_runner.result.startswith("[FAIL]"):
    # Synthesize feedback - don't copy-paste full output
    # Extract specific issues as bullet points
    retry_wave_2(changes=synthesized_feedback)
elif devops_lead.verdict == "[NEEDS_CHANGES]":
    # Minor issues - can often be fixed with direct edits
    fix_issues_or_retry_wave_2()
```

## Anti-Patterns to Avoid

1. **Don't launch script-architects and tdd-architects separately** - always in the same message
2. **Don't let tdd-architect see implementation first** - tests come from spec
3. **Don't skip the 4 Plan perspectives** - all 4 are required for Wave 1b
4. **Don't copy-paste full agent outputs** - synthesize to bullet points
5. **Don't skip devops-infra-lead review** even for "simple" changes
6. **Don't proceed to git-ops** without [PASS] from both reviewer AND test-runner
7. **Don't ignore test review** - reward hacking is a real failure mode
8. **Don't use Edit/Write for substantive changes** - delegate to bash-script-architect
9. **Don't bypass review with direct edits** - even "quick fixes" need quality gates
10. **Don't skip dependency analysis** - parallel writers can create conflicts
11. **Don't spawn tdd-architect per file** - tests are by feature, not file
12. **Don't proceed to group N+1** until group N completes

## Output Directory

`~/.claude/bash-workflow/` for intermediate artifacts:
- `explorer-findings.md`
- `plan-implementation.md`, `plan-testing.md`, `plan-security.md`, `plan-devops.md`
- `unified-plan.md`
- `review.md`

## Success Criteria

The workflow is complete when ALL are true:
1. devops-infra-lead returned `[PASS]`
2. bash-test-runner returned `[PASS]` (shellcheck clean, bats pass)
3. git-ops confirmed commit successful

**There is NO shortcut.** Even if the change is trivial, the gate is: [PASS] + [PASS] = proceed.

## Output Format

Present final summary with:
- Implementation files (absolute paths)
- Test files (absolute paths)
- Review verdict
- Next steps if needed

## References

- [Agent prompt templates](references/agent-prompts.md) - Prompt templates with placeholder variables for each wave
- [Worked examples](references/examples.md) - Multi-file task walkthrough and todo template
