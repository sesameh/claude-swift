# OUTBOX

## Overview
Cross-repository task distribution workflow that collects tasks from registered project outboxes and routes them to target project inboxes. Acts as the central message broker in the event-driven choreography system.

## Trigger
**User-Friendly**: `outbox sesame` (NOTE: no dot parameter for orchestrator workflow)
**Technical**: `OUTBOX`

**⚠️ IMPORTANT**: This is the ORCHESTRATOR-ONLY cross-repository workflow. For self-targeted tasks within a repository, use `to-inbox sesame` (universal) instead.

## Purpose
- Collect outbox tasks from all registered projects
- Distribute tasks to target project inboxes
- Enable asynchronous cross-repository communication
- Maintain audit trail of task routing operations

## Scope
**Orchestrator-Only**: This workflow is ONLY available in the claude-swift orchestrator repository. It is NOT available in registered/orchestrated projects.

## Prerequisites
- INITIALISE workflow completed (projects/ symlink exists)
- PROJECT_REGISTER registry populated with target projects
- Base repository with projects/ directory (sesameh org repositories only)
- GitHub CLI authenticated for accessing registered projects

## Security Constraint
**CRITICAL**: This workflow can ONLY be executed from sesameh organization repositories that contain a `projects/` directory. This prevents unauthorized task injection and ensures controlled distribution.

## Workflow Steps

### 1. Execution Context Validation
```
OUTBOX|step|security_check||Validate execution from authorized base repository
```

**Security Validation:**
```bash
# Use optimized Node.js audit logging
source claude/wow/scripts/audit-functions.sh

# Log workflow start
audit_log "OUTBOX" "workflow_start" "outbox_sesame" "" "Starting cross-repository task distribution workflow"

# Verify we're in a base repository with projects directory
if [ ! -d "projects" ]; then
    echo "❌ Error: OUTBOX cross-repository workflow can only be executed from base repositories"
    echo "This workflow requires a projects/ directory (sesameh org repositories only)"
    echo "Current directory: $(pwd)"
    echo ""
    echo "💡 Did you mean to run 'to-inbox sesame' instead?"
    echo "   - 'to-inbox sesame' = Process self-targeted tasks (works in any repository)"
    echo "   - 'outbox sesame' = Cross-repository distribution (orchestrator only)"
    audit_log "OUTBOX" "error" "security_check" "" "Unauthorized execution: missing projects directory"
    exit 1
fi

# Verify we have the project registry
REGISTRY_FILE="claude/project/registered-projects.json"
if [ ! -f "$REGISTRY_FILE" ]; then
    echo "Error: Project registry not found at $REGISTRY_FILE"
    echo "Run PROJECT_REGISTER workflow to register projects first"
    audit_log "OUTBOX" "error" "security_check" "" "Missing project registry file"
    exit 1
fi

echo "✓ Authorized base repository verified"
echo "✓ Project registry found"
audit_log "OUTBOX" "step" "security_check" "" "Validated execution from authorized base repository"
```

### 2. Registered Projects Discovery
```
OUTBOX|step|registry_scan||Load registered projects from registry
```

**Registry Processing:**
```bash
# Read registered projects from JSON registry
if ! command -v python3 >/dev/null 2>&1; then
    echo "Error: python3 required for JSON processing"
    audit_log "OUTBOX" "error" "registry_scan" "" "Python3 not available for JSON processing"
    exit 1
fi

# Extract registered project paths
REGISTERED_PROJECTS=$(python3 -c "
import json
with open('$REGISTRY_FILE', 'r') as f:
    data = json.load(f)
for project in data['registered_projects']:
    print(project['path'])
")

if [ -z "$REGISTERED_PROJECTS" ]; then
    echo "No registered projects found in registry"
    echo "Use 'register [org/repo] sesame' to register projects first"
    audit_log "OUTBOX" "step" "registry_scan" "" "No registered projects found - workflow completed"
    exit 0
fi

PROJECT_COUNT=$(echo "$REGISTERED_PROJECTS" | wc -l)
echo "✓ Found $PROJECT_COUNT registered projects"
audit_log "OUTBOX" "step" "registry_scan" "" "Loaded $PROJECT_COUNT registered projects from registry"
```

### 3. Outbox Collection Phase
```
OUTBOX|step|collection||Collect outbox tasks from all registered projects
```

**Task Collection:**
```bash
# Ensure base outbox directory exists
mkdir -p claude/outbox

COLLECTED_COUNT=0
echo "=== COLLECTION PHASE ==="

# Process each registered project
for PROJECT_PATH in $REGISTERED_PROJECTS; do
    if [ -d "$PROJECT_PATH/claude/outbox" ]; then
        PROJECT_NAME=$(basename "$PROJECT_PATH")
        ORG_NAME=$(basename "$(dirname "$PROJECT_PATH")")
        
        # Find outbox tasks (*.md files with timestamp format)
        OUTBOX_TASKS=$(find "$PROJECT_PATH/claude/outbox" -name "????-??-??T??-??-??-???Z_*.md" 2>/dev/null || true)
        
        if [ -n "$OUTBOX_TASKS" ]; then
            echo "Collecting from $ORG_NAME/$PROJECT_NAME:"
            TASK_COUNT=0
            
            for TASK_FILE in $OUTBOX_TASKS; do
                TASK_NAME=$(basename "$TASK_FILE")
                
                # Move task to base outbox
                if mv "$TASK_FILE" "claude/outbox/$TASK_NAME"; then
                    echo "  ✓ Collected: $TASK_NAME"
                    ((COLLECTED_COUNT++))
                    ((TASK_COUNT++))
                else
                    echo "  ❌ Failed to collect: $TASK_NAME"
                    audit_log "OUTBOX" "error" "collection" "$TASK_NAME" "Failed to collect task from $ORG_NAME/$PROJECT_NAME"
                fi
            done
            
            if [ $TASK_COUNT -gt 0 ]; then
                audit_log "OUTBOX" "step" "collection" "$ORG_NAME/$PROJECT_NAME" "Collected $TASK_COUNT tasks from registered project"
            fi
        fi
    fi
done

echo "✓ Collection complete: $COLLECTED_COUNT tasks collected"
audit_log "OUTBOX" "step" "collection" "" "Collection phase completed: $COLLECTED_COUNT tasks collected from registered projects"
```

### 4. Task Distribution Phase
```
OUTBOX|step|distribution||Distribute tasks to target project inboxes
```

**Task Routing:**
```bash
echo "=== DISTRIBUTION PHASE ==="

# Find all tasks in base outbox
BASE_OUTBOX_TASKS=$(find claude/outbox -name "????-??-??T??-??-??-???Z_*.md" 2>/dev/null | sort || true)

if [ -z "$BASE_OUTBOX_TASKS" ]; then
    echo "No tasks to distribute"
    audit_log "OUTBOX" "step" "distribution" "" "No tasks found for distribution - workflow completed"
    exit 0
fi

TASK_COUNT=$(echo "$BASE_OUTBOX_TASKS" | wc -l)
echo "Found $TASK_COUNT tasks for distribution"

DISTRIBUTED_COUNT=0
FAILED_COUNT=0

for TASK_FILE in $BASE_OUTBOX_TASKS; do
    TASK_NAME=$(basename "$TASK_FILE")
    
    # Extract target repository from filename: TIMESTAMP_target-repo_task-name.md
    TARGET_REPO=$(echo "$TASK_NAME" | sed 's/^[^_]*_\([^_]*\)_.*$/\1/')
    
    if [ -z "$TARGET_REPO" ]; then
        echo "✗ Invalid task filename format: $TASK_NAME"
        audit_log "OUTBOX" "error" "distribution" "$TASK_NAME" "Invalid filename format - cannot extract target repository"
        ((FAILED_COUNT++))
        continue
    fi
    
    # Handle self-targeted tasks (target same as base repository)
    BASE_REPO_NAME=$(basename "$(pwd)")
    if [ "$TARGET_REPO" = "$BASE_REPO_NAME" ]; then
        # Self-targeted task - route to base repository's inbox
        TARGET_PATH="."
    else
        # Find target project path in registry
        TARGET_PATH=$(python3 -c "
import json
with open('$REGISTRY_FILE', 'r') as f:
    data = json.load(f)
for project in data['registered_projects']:
    repo_name = project['repository'].split('/')[-1]
    if repo_name == '$TARGET_REPO':
        print(project['path'])
        break
" 2>/dev/null)
    fi
    
    if [ -z "$TARGET_PATH" ] || [ ! -d "$TARGET_PATH" ]; then
        echo "✗ Target repository '$TARGET_REPO' not found or inaccessible"
        echo "  Task remains in outbox: $TASK_NAME"
        audit_log "OUTBOX" "error" "distribution" "$TASK_NAME" "Target repository '$TARGET_REPO' not found in registry"
        ((FAILED_COUNT++))
        continue
    fi
    
    # Ensure target inbox exists
    mkdir -p "$TARGET_PATH/claude/inbox"
    
    # Move task to target inbox
    if mv "$TASK_FILE" "$TARGET_PATH/claude/inbox/$TASK_NAME"; then
        echo "✓ Delivered to $TARGET_REPO: $TASK_NAME"
        audit_log "OUTBOX" "step" "distribution" "$TASK_NAME" "Successfully delivered task to $TARGET_REPO"
        ((DISTRIBUTED_COUNT++))
    else
        echo "✗ Failed to deliver to $TARGET_REPO: $TASK_NAME"
        audit_log "OUTBOX" "error" "distribution" "$TASK_NAME" "Failed to move task to $TARGET_REPO inbox"
        ((FAILED_COUNT++))
    fi
done

echo "✓ Distribution complete: $DISTRIBUTED_COUNT delivered, $FAILED_COUNT failed"
audit_log "OUTBOX" "step" "distribution" "" "Distribution phase completed: $DISTRIBUTED_COUNT delivered, $FAILED_COUNT failed"
```

### 5. Distribution Summary
```
OUTBOX|step|completion_summary||Provide outbox workflow completion summary
```

**Completion Report:**
```bash
echo ""
echo "=== OUTBOX WORKFLOW COMPLETE ==="
echo "Collection: $COLLECTED_COUNT tasks gathered from registered projects"
echo "Distribution: $DISTRIBUTED_COUNT tasks delivered successfully"
if [ $FAILED_COUNT -gt 0 ]; then
    echo "Failures: $FAILED_COUNT tasks remain in outbox (see errors above)"
    echo "Check target repositories and retry outbox workflow"
    audit_log "OUTBOX" "workflow_complete" "outbox_sesame" "" "OUTBOX workflow completed with failures: $COLLECTED_COUNT collected, $DISTRIBUTED_COUNT distributed, $FAILED_COUNT failed"
else
    echo "Status: All tasks distributed successfully"
    audit_log "OUTBOX" "workflow_complete" "outbox_sesame" "" "OUTBOX workflow completed successfully: $COLLECTED_COUNT collected, $DISTRIBUTED_COUNT distributed"
fi
echo "Registry: $PROJECT_COUNT registered projects processed"
echo ""
echo "Next steps:"
echo "- Target repositories can run 'inbox sesame' to process received tasks"
echo "- Use 'next sesame' to see recommended issues after task processing"
```

## Task File Format

**Filename Convention**: `YYYY-MM-DDTHH-MM-SS-sssZ_target-repo_task-name.md`
- **Timestamp**: Full ISO 8601 with milliseconds for precise ordering
- **Target Repo**: Repository name (not org/repo, just repo name)
- **Task Name**: Descriptive identifier with hyphens

**Example Filenames**:
```
2025-07-14T15-30-45-123Z_claude-swift_update-workflows.md
2025-07-14T15-31-02-456Z_splectrum_migrate-to-v2.md
2025-07-14T15-31-15-789Z_spl1_workflow-refresh.md
```

**Task Content Structure**:
```markdown
---
source: jules-tenbos/splectrum
target: sesameh/claude-swift
created: 2025-07-14T15:30:45.123Z
priority: normal
type: workflow-update
---

# Task: Update to Latest Workflows

## Description
Update target repository to use latest v2 workflow patterns.

## Requirements
- [ ] Update SESSION_START workflow
- [ ] Add MANDATORY_RULES_REFRESH workflow
- [ ] Test all sesame triggers

## Context
Following claude-swift v1.1.0 release, workflow synchronization needed.
```

## Error Handling

### Invalid Execution Context
```bash
if [ ! -d "projects" ]; then
    echo "Error: OUTBOX workflow requires base repository with projects/ directory"
    echo "This security constraint prevents unauthorized task distribution"
    exit 1
fi
```

### Missing Project Registry
```bash
if [ ! -f "$REGISTRY_FILE" ]; then
    echo "Error: Project registry not found"
    echo "Run PROJECT_REGISTER to register projects first"
    exit 1
fi
```

### Target Repository Not Found
```bash
if [ ! -d "$TARGET_PATH" ]; then
    echo "Warning: Target repository '$TARGET_REPO' not accessible"
    echo "Task remains in outbox for manual handling"
    # Task file remains in outbox for retry
fi
```

### Collection Failures
```bash
if ! mv "$TASK_FILE" "claude/outbox/$TASK_NAME"; then
    echo "Warning: Failed to collect task from $PROJECT_PATH"
    echo "Check file permissions and disk space"
    # Original task remains in source outbox
fi
```

## Success Criteria
- All registered projects scanned for outbox tasks
- Tasks successfully collected into base outbox
- Tasks routed to correct target project inboxes
- Failed deliveries remain in base outbox with clear error messages
- Complete audit trail of all operations

## Integration Points
- **PROJECT_REGISTER**: Uses registry to discover registered projects
- **INBOX**: Target workflow for processing delivered tasks
- **AUDIT_LOGGING**: Log all task collection and distribution operations
- **Security**: Restricted to sesameh organization base repositories

## Usage Examples

### Distribute all pending tasks
```bash
# Run from sesameh/claude-swift or other base repository
outbox sesame
# Result: Collects from all registered projects, distributes to targets
```

### Error scenarios
```bash
# From non-base repository
outbox sesame
# Result: Error - only base repositories can run OUTBOX

# No registered projects
outbox sesame  
# Result: No tasks to process, suggests registering projects
```

## Directory Structure

**Before OUTBOX**:
```
sesameh/claude-swift/claude/outbox/          # Base outbox (empty)
projects/org1/repo1/claude/outbox/           # Has pending tasks
projects/org2/repo2/claude/outbox/           # Has pending tasks
```

**After OUTBOX**:
```
sesameh/claude-swift/claude/outbox/          # Empty (all distributed)
projects/org1/repo1/claude/outbox/           # Empty (collected)
projects/org2/repo2/claude/outbox/           # Empty (collected)
projects/target-org/target-repo/claude/inbox/  # Received tasks
```

## Trigger Pattern

```markdown
**OUTBOX** → See [claude/project/workflows/OUTBOX.md](./claude/project/workflows/OUTBOX.md)
```

Use when:
- Distributing cross-repository tasks
- Centralizing task routing operations
- Implementing event-driven choreography
- Processing accumulated outbox tasks from multiple projects