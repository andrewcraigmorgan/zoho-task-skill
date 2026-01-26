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

### Status: "To do"
- Reads task description and comments
- Explores codebase to understand requirements
- Creates implementation todo list
- Implements changes and runs tests
- Creates feature branch and BitBucket PR
- Updates task status to "Open" with PR link

### Status: "Open" or "In Review"
- Creates Playwright E2E test for screenshots
- Captures evidence of the feature working
- Uploads screenshots to Zoho task
- Posts client-friendly comment with embedded images

## Requirements

- [Zoho Projects MCP Server](https://github.com/example/zoho-projects-mcp) configured
- [Bitbucket MCP Server](https://github.com/example/bitbucket-mcp) configured (for PR creation)

## Configuration

The skill contains project-specific IDs that you'll need to update for your own Zoho Projects setup:
- Project ID
- Status IDs
- BitBucket repository details

## License

MIT
