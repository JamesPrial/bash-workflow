# Examples and Templates

## Todo Template

```
1. Create task breakdown for {FEATURE}
2. Wave 1a: Launch Explore agent (dependency graph + features)
3. Wave 1b: Launch 4 Plan agents in parallel
4. Synthesize unified plan from 4 perspectives
5. Wave 2 Group 1: Launch [N architects + M testers] for group 1 (TOGETHER)
6. Wave 2 Group 2+: Repeat for remaining groups (sequential)
7. Wave 3: Launch devops-infra-lead + bash-test-runner
8. Process verdict
9. Complete workflow / iterate if needed
10. Git operations (if approved)
```

## Example: Multi-File Task

Task: "Add backup functionality with rotation to the sync scripts"

**Wave 1a - Explorer output:**
```
## Files
| Path | Action | Group | Notes |
|------|--------|-------|-------|
| scripts/lib/backup.sh | create | 1 | Backup functions (no deps) |
| scripts/sync.sh | modify | 2 | Add backup before sync |
| scripts/cleanup.sh | create | 2 | Rotation logic |

## Features
### Feature: Backup Creation
- Files: [scripts/lib/backup.sh, scripts/sync.sh]
- Behaviors:
  - Create timestamped backup before sync
  - Skip backup if no changes detected

### Feature: Backup Rotation
- Files: [scripts/cleanup.sh]
- Behaviors:
  - Keep last N backups (configurable)
  - Remove oldest when limit exceeded
```

**Wave 1b - 4 Plan agents (single message with 4 Task calls):**
```
Plan (Implementation): Focus on script structure
Plan (Testing): Edge cases for backup/rotation
Plan (Security): Permission handling for backup files
Plan (DevOps): Cross-platform tar/date commands
```

**Wave 2 execution:**
```
Group 1 (single message with 2 Task calls):
  - bash-script-architect: scripts/lib/backup.sh
  - bash-tdd-architect: "Backup Creation" feature
  [wait for completion]

Group 2 (single message with 4 Task calls):
  - bash-script-architect: scripts/sync.sh
  - bash-script-architect: scripts/cleanup.sh
  - bash-tdd-architect: "Backup Rotation" feature
  - bash-tdd-architect: "Sync Integration" feature (if identified)
  [wait for completion]
```

**Wave 3 (single message with 2 Task calls):**
```
  - bash-test-runner: Run shellcheck + bats
  - devops-infra-lead: Review all code and tests
  [wait for both, check verdicts]
```
