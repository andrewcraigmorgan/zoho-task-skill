# Zoho Task Skill for Claude Code

A Claude Code skill that helps work on Zoho Projects tasks based on their current status.

## Installation

Copy the `SKILL.md` file to your Claude Code skills directory:

```bash
# Global installation (available in all projects)
mkdir -p ~/.claude/skills/zoho-task
cp SKILL.md ~/.claude/skills/zoho-task/

# Or project-level installation
mkdir -p .claude/skills/zoho-task
cp SKILL.md .claude/skills/zoho-task/
```

## Usage

```
/zoho-task <task-reference>
```

The task reference can be:
- **Full URL**: `https://projects.zoho.com/portal/...`
- **Task ID**: `1013893000023483027`
- **Task prefix**: `CA6-T176`

## What It Does

This skill fetches a Zoho Projects task and routes to different workflows based on status:

### Status: "Awaiting Approval" or "Need more information" (Scoping/Clarification)
- Reads task description and existing comments
- Explores codebase to understand technical considerations
- Identifies gaps, ambiguities, and decision points
- Generates 5-8 targeted clarification questions
- Posts questions to Zoho (after user approval)
- Keeps status unchanged until client responds

### Status: "To do" (Implementation)
- Reads task description and comments
- Explores codebase to understand requirements
- Creates implementation todo list
- Implements changes and runs tests
- Creates feature branch and BitBucket PR
- Updates task status to "Open" with PR link

### Status: "Open" or "In Review" (Documentation)
- Creates Playwright E2E test for screenshots
- Captures evidence of the feature working
- Uploads screenshots to Zoho task
- Posts client-friendly comment with embedded images

## Requirements

- [Zoho Projects MCP Server](https://github.com/andrewcraigmorgan/zoho-projects-mcp) configured
- [Bitbucket MCP Server](https://github.com/aashari/mcp-server-atlassian-bitbucket) configured (for PR creation)

## Configuration

Each project using this skill needs configuration in its `.claude/CLAUDE.md` file. This tells the skill which Zoho project, BitBucket repo, and production URL to use.

**First-time setup**: If configuration is missing, the skill will prompt you for each value and offer to save them to `.claude/CLAUDE.md` automatically.

### Manual Setup

1. Create `.claude/CLAUDE.md` in your project root (if it doesn't exist)
2. Add the following configuration:

```markdown
# Zoho Projects Configuration
- **Project ID**: `1013893000022796035`
- **Open Status ID**: `1013893000001076068`

# BitBucket Configuration
- **Workspace**: `myworkspace`
- **Repository**: `my-repo-slug`

# URLs
- **Production URL**: `https://myapp.example.com`
```

### Finding Your IDs

**Zoho Project ID**: Found in the URL when viewing your project in Zoho Projects.

**Zoho Status IDs**: Ask Claude to run:
```
mcp__zoho-projects__list_statuses(project_id: "<PROJECT_ID>")
```

**BitBucket Workspace/Repo**: Found in your repository URL: `bitbucket.org/<workspace>/<repo-slug>`

## License

MIT
