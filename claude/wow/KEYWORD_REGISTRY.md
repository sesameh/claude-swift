# KEYWORD_REGISTRY

This file maintains the complete registry of uppercase workflow trigger keywords used in the CLAUDE.md system.

## Custom Workflow Keywords

| Keyword | File | Purpose | Trigger Context | Sesame Alias |
|---------|------|---------|-----------------|--------------|
| **SESSION_START** | `SESSION_START.md` | Time tracking & session initiation | Every new Claude session | `start sesame` |
| **SESSION_END** | `SESSION_END.md` | Session termination & learning capture | End of Claude session | `finish sesame` |
| **GIT_WORKFLOW** | `GIT_WORKFLOW.md` | Branching, commits, releases | Git operations, version releases | `git sesame` |
| **OPERATIONAL_RULES** | `OPERATIONAL_RULES.md` | Development rules framework | Development decisions, coding standards | `rules sesame` |
| **RELEASE_PROCESS** | `RELEASE_PROCESS.md` | Complete release workflow with log archiving | Creating platform releases, version closure | `release sesame`, `patch sesame` |
| **NEXT_ISSUE** | `NEXT_ISSUE.md` | Automated next issue selection workflow | Determining which GitHub issue to work on next | `next sesame` |
| **VERSION_TRANSITION** | `VERSION_TRANSITION.md` | Mandatory post-release knowledge processing and next version preparation | After RELEASE_PROCESS completion | `transition sesame` |
| **NEW_VERSION_PLANNING** | `NEW_VERSION_PLANNING.md` | Next version planning and roadmap development | Version planning, feature prioritization | `version sesame` |
| **DOCUMENTATION_WORKFLOW** | `DOCUMENTATION_WORKFLOW.md` | Documentation standards and compliance management | Documentation creation, maintenance, compliance | `docs sesame` |
| **MANDATORY_RULES_REFRESH** | `MANDATORY_RULES_REFRESH.md` | Systematic scanning and validation of all mandatory rules | Rule compliance, violation detection, corrective actions | `mandatory sesame` |
| **COMMIT** | `COMMIT.md` | Intelligent commit workflow with automatic issue closure | Change staging, commit message generation, issue management | `commit sesame` |
| **ISSUE_CACHE** | `ISSUE_CACHE.md` | Local cache management for GitHub issues and milestones | Performance optimization, reduced API calls, offline access | `issue sesame` |
| **CREATE_ISSUE** | `CREATE_ISSUE.md` | Manual issue creation with standardized metadata structure | Ad-hoc issue creation, structured backlog management | `issue sesame` |
| **INBOX** | `INBOX.md` | Task ingestion workflow converting inbox tasks to GitHub issues | Cross-repository task processing, task-to-issue conversion, inbox management | `inbox sesame` |
| **TO_INBOX** | `OUTBOX.md` | Self-targeted task processing from outbox to inbox | Local task queuing, outbox-to-inbox movement, task processing | `to-inbox sesame` |
| **TASK_CREATE** | `TASK_CREATE.md` | Interactive task creation workflow for cross-repository communication | Guided task creation, metadata collection, standardized formatting | `task [org/repo] sesame`, `task . sesame` |

## Built-in Claude Code Keywords

These represent my automatic behaviors and built-in workflows:

| Keyword | Built-in Behavior | Trigger Context |
|---------|------------------|-----------------|
| **TODO_MANAGEMENT** | Automatic TodoWrite/TodoRead usage for task tracking | Complex multi-step tasks, planning |
| **TOOL_SELECTION** | Strategic tool choice (Task vs Read/Grep/Glob) | Codebase exploration vs specific file access |
| **SEARCH_STRATEGY** | Parallel search operations, context-aware searching | Finding files, functions, or patterns |
| **ERROR_HANDLING** | Retry logic, fallback strategies, error analysis | Tool failures, unexpected results |
| **CONCURRENCY** | Batch multiple tool calls in single response | Information gathering, parallel operations |
| **CODE_ANALYSIS** | Read-first approach, pattern recognition, convention following | Understanding codebases, making changes |
| **COMMIT_WORKFLOW** | git status + diff + log before commits, staging all files | Git operations, pre-commit checks |
| **RESPONSE_CONCISENESS** | Minimize output tokens, direct answers | All communication |

## Context-Specific Keywords (Future)

Potential additional workflow triggers:

| Context | Potential Keywords | Purpose |
|---------|-------------------|---------|
| Debugging | DEBUG_WORKFLOW | Systematic debugging approach |
| Testing | TESTING_WORKFLOW | Test creation and execution |
| Documentation | DOCS_WORKFLOW | Documentation standards |
| Learning | LEARNING_CAPTURE | End-of-session learning documentation |

## Sesame Alias System

The **sesame suffix** provides user-friendly triggers for workflow keywords:

- **Pattern**: `[descriptive_word] sesame` → `WORKFLOW_KEYWORD`
- **Magic word**: "sesame" (reference to "Open Sesame" - opens workflow execution)
- **Usage**: Users type friendly aliases, system maps to technical keywords
- **Timelog**: Always records technical keywords for consistency

### Sesame Alias Mapping

```
start sesame     → SESSION_START
finish sesame    → SESSION_END  
git sesame       → GIT_WORKFLOW
rules sesame     → OPERATIONAL_RULES
release sesame   → RELEASE_PROCESS (full version)
patch sesame     → RELEASE_PROCESS (patch mode)
next sesame      → NEXT_ISSUE
mandatory sesame → MANDATORY_RULES_REFRESH
issue sesame     → CREATE_ISSUE
inbox sesame     → INBOX
to-inbox sesame  → TO_INBOX
task sesame      → TASK_CREATE
```

## Registry Maintenance Rules

1. **Add new keywords here FIRST** before creating workflow files
2. **Update this registry** when modifying existing workflows
3. **Check for conflicts** - ensure keywords are unique and clear
4. **Follow naming convention**: NOUN_VERB or CONTEXT_ACTION format
5. **Keep alphabetical order** within each section for easy scanning
6. **Add sesame alias** for all new workflow keywords using descriptive, memorable words

## Usage Pattern

```markdown
**KEYWORD_NAME** → See [workflows/FILENAME.md](./workflows/FILENAME.md)
```

Example:
```markdown
**SESSION_START** → See [workflows/SESSION_START.md](./workflows/SESSION_START.md)
```