# Agent Prompt Templates

These are prompt templates for the Task tool. Placeholders like `{TASK}`, `{EXPLORER_SUMMARY}`, `{FILE_PATH}` must be replaced with actual values when launching each agent.

## Wave 1a: Explore Agent

```
Analyze codebase for: {TASK}

Find:
- Relevant shell scripts and their dependencies
- Existing patterns and conventions (set -euo pipefail, function patterns)
- Test locations (test/*.bats, *.bats files)
- Platform considerations (macOS vs Linux specifics)

Output to ~/.claude/bash-workflow/explorer-findings.md with this structure:

## Files
| Path | Action | Group | Notes |
|------|--------|-------|-------|
| scripts/sync.sh | modify | 1 | Entry point |
| scripts/lib/utils.sh | create | 1 | Helper functions |
| scripts/backup.sh | modify | 2 | Depends on utils.sh |

Group assignment rules:
- Group 1: Files with no dependencies on other modified files
- Group N+1: Files that depend on files in group N
- Files in same group can be written in parallel

## Features
### Feature: {name}
- **Files**: [list of files involved]
- **Behaviors**:
  - {behavior 1}
  - {behavior 2}
```

## Wave 1b: Plan Agents (4 in parallel)

### Plan Agent (Implementation Perspective)

```
Design implementation approach for: {TASK}

Based on exploration findings: {EXPLORER_SUMMARY}

Focus on:
- Script structure and organization
- Function decomposition
- Dependencies between components
- Reusable patterns from codebase

Output implementation design to ~/.claude/bash-workflow/plan-implementation.md
```

### Plan Agent (Testing Perspective)

```
Design test strategy for: {TASK}

Based on exploration findings: {EXPLORER_SUMMARY}

Focus on:
- Behaviors to verify with bats tests
- Edge cases: empty input, missing files, bad permissions, signals
- Error paths and failure modes
- Platform-specific test considerations

Output test strategy to ~/.claude/bash-workflow/plan-testing.md
```

### Plan Agent (Security Perspective)

```
Analyze security considerations for: {TASK}

Based on exploration findings: {EXPLORER_SUMMARY}

Focus on:
- Input validation requirements
- Permission handling (file modes, sudo usage)
- Secret handling (no hardcoded secrets, use env vars)
- Command injection risks
- Unsafe patterns to avoid

Output security analysis to ~/.claude/bash-workflow/plan-security.md
```

### Plan Agent (DevOps Perspective)

```
Analyze operational considerations for: {TASK}

Based on exploration findings: {EXPLORER_SUMMARY}

Focus on:
- macOS vs Linux compatibility (BSD vs GNU tools)
- CI/CD integration considerations
- Deployment and installation
- Dependency requirements (external commands needed)
- Idempotency for repeated runs

Output DevOps analysis to ~/.claude/bash-workflow/plan-devops.md
```

## Wave 2: Implementation Agents

### bash-script-architect Agent (per-file)

```
Implement changes to: {FILE_PATH}

Task context:
{OVERALL_TASK_DESCRIPTION}

Unified plan summary:
{UNIFIED_PLAN_SUMMARY}

Scope for this file:
{FILE_SPECIFIC_CHANGES}

Context from exploration:
- Group: {GROUP_NUM} (files in this group execute in parallel)
- Related files in same group: {SIBLING_FILES}
- Interface contracts to implement: {INTERFACES}

Requirements:
- Follow bash-script-architect standards (set -euo pipefail, quoted variables)
- Include function documentation headers
- Handle errors with meaningful messages
- DO NOT write tests (bash-tdd-architect handles this)
- If defining interfaces other files depend on, make them clear

Output: Write implementation for {FILE_PATH} only
```

### bash-tdd-architect Agent (per-feature)

```
Write bats tests for feature: {FEATURE_NAME}

Behaviors to verify:
{BEHAVIOR_LIST}

Files involved: {FILE_LIST}

Security considerations from plan:
{SECURITY_NOTES}

Requirements:
- Design tests from SPEC, not by reading implementation
- Use bats-core with bats-assert and bats-support
- Cover happy paths, edge cases, and error conditions
- Test on both macOS and Linux if platform-specific code
- One test file per feature: test_{feature_slug}.bats

Test file location: test/ or match existing project convention

Output: Write comprehensive bats test file for this feature
```

## Wave 3: Review and Test Agents

### devops-infra-lead Agent (Reviewer)

```
Review implementation for: {TASK}

Files changed: {FILE_LIST}

Review using the Bash Code Review Checklist and Bats Test Review Checklist.

Check code for:
- set -euo pipefail usage
- Quoted variable expansions
- Error handling completeness
- Cross-platform compatibility
- Shellcheck compliance

Check tests for:
- Behavior-focused (not implementation-focused)
- Edge case coverage
- Test independence
- Meaningful assertions

Return verdict:
- [PASS]: Ready to commit
- [FAIL]: List test failures and critical issues
- [NEEDS_CHANGES]: List issues requiring fixes
```

### bash-test-runner Agent

```
Run shellcheck and bats tests for the project.

Return:
- [PASS] if all shellcheck clean and tests pass
- [FAIL]: [list of shellcheck errors and test failures]
- [ERROR]: [setup issues like missing bats]
```

## Wave 4: Git Agent

### git-ops Agent

```
Commit and push: [summary of what was implemented]
```
