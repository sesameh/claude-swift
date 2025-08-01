[← Back to Workflows](../workflows/) | [← Back to Claude-Swift Home](../../../README.md)

# COMMIT Workflow

## Overview
Intelligent commit workflow that stages changes, creates descriptive commit messages, and automatically closes GitHub issues if work resolves them.

## Trigger
**User-Friendly**: `commit sesame`
**Technical**: `COMMIT`
**SESSION_END Integration**: Automatically called during SESSION_END step 3

## Purpose
- Stage all current changes for commit
- Generate intelligent commit message based on changes and context
- Detect if work resolves any GitHub issues
- Commit changes with proper attribution
- Push to main branch
- Close resolved issues automatically

## Dual Context Support
- **Manual Usage**: `commit sesame` for mid-session commits
- **SESSION_END Integration**: Automatic execution during session closure with session-aware messaging

## Prerequisites
- Clean working directory or meaningful changes
- GitHub CLI (`gh`) configured for issue management
- Current work represents a logical commit unit

## Workflow Steps

### 1. Initialize Audit Logging
```bash
# Load Node.js audit functions
# Automated audit logging - no manual sourcing required

# Start workflow with explicit logging
claude/wow/scripts/audit-manage log "COMMIT" "workflow_start" "commit_sesame" "" "Initiated COMMIT workflow for session work completion"
```

### 2. Change Assessment
```bash
claude/wow/scripts/audit-manage log "COMMIT" "step" "change_assessment" "" "Analyzing current changes and git status"
```

**Actions:**
1. Run `claude/wow/scripts/git-manage status` to get comprehensive repository status
2. Analyze changed files, staged/unstaged status, and scope of changes
3. Check for untracked files that should be included
4. Validate changes represent a logical commit unit

### 3. Issue Detection
```bash
claude/wow/scripts/audit-manage log "COMMIT" "step" "issue_detection" "" "Scanning changes and context for resolved issues"
```

**Actions:**
1. Check recent audit log entries for issue references
2. Search commit message context for issue numbers (#XX)
3. Analyze changed files against open GitHub issues
4. Identify issues that this work resolves

### 4. Commit Message Generation
```bash
claude/wow/scripts/audit-manage log "COMMIT" "step" "message_generation" "" "Generating descriptive commit message"
```

**Format varies by context:**

**Manual Commit (`commit sesame`):**
```
Brief summary of changes

- Detailed implementation point
- Another key change
- Reference to methodology or approach used

[Context: Why this change was needed]
[Closes #XX] (if issue resolved)

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**SESSION_END Integration:**
```
Session complete: [session summary]

- Key accomplishments from session
- Major changes implemented
- Issues resolved during session

[Session context and learnings]
[Closes #XX, #YY] (if issues resolved)

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### 5. Commit Execution
```bash
claude/wow/scripts/audit-manage log "COMMIT" "step" "commit_execution" "" "Executing automated commit workflow"
```

**Actions:**
```bash
# Execute automated commit script with generated message
claude/wow/scripts/git-manage commit --message "Generated commit message"

# The commit script automatically handles:
# - Staging all changes (git add .)
# - Creating commit with message
# - Pushing to main branch
# - Audit logging throughout process
# - Issue detection and closure
# - Error handling and recovery
```

### 6. Issue Closure
```bash
claude/wow/scripts/audit-manage log "COMMIT" "step" "issue_closure" "" "Automated issue closure handled by commit script"
```

**Actions:**
The automated commit script handles issue closure automatically:
1. **Issue Detection**: Scans commit context and recent audit logs for resolved issues
2. **GitHub API Integration**: Uses `claude/wow/scripts/issue-manage` to close issues
3. **Cache-first closure**: Updates local cache, then syncs to GitHub
4. **Audit logging**: Records all issue closures with commit references

**Manual issue closure** (if needed):
```bash
# Close specific issue with commit reference
claude/wow/scripts/issue-manage close XX
```

### 7. Workflow Completion
```bash
claude/wow/scripts/audit-manage log "COMMIT" "workflow_complete" "commit_sesame" "" "COMMIT workflow completed successfully - changes committed and issues resolved"
```

## Smart Detection Patterns

### Issue Reference Detection
- **Audit log scanning**: Look for recent `#XX` references
- **File change analysis**: Match changed files to issue descriptions
- **Commit context**: Analyze work done against open issues
- **User confirmation**: Always confirm before closing issues

### Commit Message Intelligence
- **Change scope**: Single file vs multiple files
- **Change type**: feat/fix/docs/refactor based on changes
- **Implementation details**: Extract key changes from diffs
- **Context preservation**: Include why change was needed

## Error Handling

The `commit` script handles common scenarios automatically:
- **No changes**: Exits gracefully with status message
- **Merge conflicts**: Provides clear guidance on resolution steps
- **Push failures**: Attempts sync and retry with recovery options

## Integration Points

### SESSION_END Integration
- **Automatic Execution**: COMMIT workflow is automatically called during SESSION_END step 3
- **Session-Aware Messaging**: Generates session summary commits instead of generic messages
- **Complete Closure**: Ensures all session work is committed and issues are properly closed
- **Zero Manual Effort**: Issue tracking happens automatically at session end

### Issue Workflow Integration
- Detects when work resolves GitHub issues
- Provides clean closure with commit references
- Maintains issue tracking accuracy

### Audit Logging Integration
- All commit operations logged in audit trail
- Issue closures recorded for accountability
- Commit hashes preserved for reference

## Success Criteria
- All changes staged and committed successfully
- Descriptive commit message generated
- Resolved issues automatically closed
- Changes pushed to main branch
- Audit trail complete

## Usage Examples

### Simple Commit
```
`commit sesame`
# Result: Analyzes changes, generates message, commits and pushes
```

### With Issue Resolution
```
# After working on issue #XX
`commit sesame`
# Result: Commits changes and closes issue #XX automatically
```

### Multiple Issues
```
# After fixing bugs #XX and #YY
`commit sesame`
# Result: Commits changes and closes both issues with references
```

## Benefits
- **Eliminates manual commit composition**
- **Ensures consistent commit format**
- **Automatically manages issue lifecycle**
- **Provides intelligent change detection**
- **Maintains clean git history**
- **Integrates with existing workflows**

---

*Streamlined commit workflow optimized for AI-assisted development with automatic issue management.*