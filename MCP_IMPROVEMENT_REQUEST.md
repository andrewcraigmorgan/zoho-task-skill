# MCP Tool Improvement Request: Zoho Comment Validation

**Date:** 2026-01-29
**Issue:** Comments posted to Zoho without user approval

## Problem

The `add_task_comment` and `edit_task_comment` MCP tools accept empty or whitespace-only content parameters without validation. This allows accidental posting of empty comments or posting without proper approval flow being followed.

## Requested Changes

### 1. Content Validation

The MCP tools should reject requests with empty or whitespace-only content:

```typescript
// In add_task_comment handler
if (!content || content.trim() === '') {
  throw new Error('Comment content cannot be empty')
}
```

### 2. Optional Confirmation Parameter

Add an optional `confirmed` parameter that skills/prompts can use to enforce approval workflows:

```typescript
interface AddTaskCommentParams {
  project_id: string
  task_id: string
  content: string
  confirmed?: boolean  // If skill requires confirmation, this must be true
}
```

### 3. Content Length Minimum

Require a minimum content length to prevent accidental submissions:

```typescript
if (content.trim().length < 10) {
  throw new Error('Comment content too short (minimum 10 characters)')
}
```

## Impact

Without these changes:
- Empty comments can be posted to client-facing Zoho tasks
- No technical guardrail prevents posting without approval
- Relies entirely on prompt/skill instructions being followed correctly

## Workaround

Currently relying on skill instructions and CLAUDE.md reminders to enforce approval workflow. This is not foolproof as the AI can still call the tool without following the process.
