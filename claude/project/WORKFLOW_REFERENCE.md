# Workflow Reference - Claude Project Workflows

Reference guide to project-specific workflows that extend the base Claude Ways of Working system.

## Project-Specific Workflows

| Workflow | File | Purpose |
|----------|------|---------|
| INITIALISE | `workflows/INITIALISE.md` | One-time workspace setup and multi-project configuration |
| PROJECT_REGISTER | `workflows/PROJECT_REGISTER.md` | Automated project registration with symlink creation |

**Note**: OUTBOX workflow was moved to `../wow/workflows/OUTBOX.md` during workflow consolidation.

## Base WoW Workflows

**Important**: This project inherits all base workflows from the Claude Ways of Working system.

For complete workflow reference, see: `../wow/WORKFLOW_REFERENCE.md`

**Available base workflows include:**
- Session management (SESSION_START, SESSION_END)
- Development workflows (COMMIT, GIT_WORKFLOW, DOCUMENTATION_WORKFLOW, OPERATIONAL_RULES)
- Issue management (ISSUE, CREATE_ISSUE, CLOSE_ISSUE, NEXT_ISSUE, ISSUE_CACHE)
- Release management (RELEASE, FULL_RELEASE, PATCH_RELEASE, VERSION_TRANSITION, NEW_VERSION_PLANNING, VERSION)
- Task management (INBOX, OUTBOX, TASK_CREATE)

## Workflow Inheritance

### Extension Pattern
Project workflows extend but do not override WoW workflows:
- **Project-specific needs**: Handled by workflows in this directory
- **Base functionality**: Inherited from `../wow/workflows/`
- **Trigger integration**: Project triggers work alongside WoW triggers

### Script Integration
Project workflows may use:
- **WoW scripts**: `../wow/scripts/` for base functionality
- **Project scripts**: `scripts/` for project-specific implementation (if needed)

## Usage

**Trigger Information**: See `../CLAUDE.md` for user-friendly sesame triggers  
**Base Workflows**: See `../wow/WORKFLOW_REFERENCE.md` for inherited workflows  
**WoW Scripts**: See `../wow/SCRIPT_REFERENCE.md` for available implementation scripts

---

*Project-specific workflow extensions to the Claude Ways of Working system*