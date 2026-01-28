---
name: zoho-task
description: Work on a Zoho Projects task based on its current status. Accepts URL, ID, or prefix (e.g., CA6-T238).
---

# Zoho Task Skill

Work on a Zoho Projects task based on its current status.

## Important: Non-Technical Client

The client reviewing Zoho updates is **non-technical**. All comments, updates, and documentation must be:

- **Jargon-free**: No technical terms (avoid "API", "component", "endpoint", "migration", "Vue", "Laravel", "database", "query", etc.)
- **Plain English**: Write as if explaining to someone unfamiliar with software development
- **Action-focused**: Describe what users can now DO, not how it was built
- **Scannable**: Use short sentences, bullet points, and clear headings

### Writing Style Examples

| Technical (Avoid) | Client-Friendly (Use) |
|-------------------|----------------------|
| "Added Vue component for CSV export" | "Added a button to download data as a spreadsheet" |
| "Fixed API endpoint returning 500 error" | "Fixed an issue where the page sometimes failed to load" |
| "Implemented database migration for new field" | "Added ability to track additional information" |
| "Refactored authentication middleware" | "Improved the login process" |
| "Updated TypeScript interfaces" | (Don't mention - internal change) |
| "PR merged to staging branch" | "Changes are ready for review" |

### What to Include vs Exclude

**Include:**
- What the user can now see or do
- Where to find the new feature (with links)
- Screenshots showing the feature in action
- Simple steps to verify it works

**Exclude:**
- Technical implementation details
- Code changes, file names, or branch names (keep in separate "Code Changes" section for internal reference)
- Framework or library names
- Database or architecture changes

## Usage

```
/zoho-task <task-reference>
```

The task reference can be:

- **Full URL**: `https://projects.zoho.com/portal/mtcmedialtd#milestone/.../task-detail/1013893000023483027`
- **Task ID**: `1013893000023483027`
- **Task prefix/code**: `CA6-T176`

Examples:

```
/zoho-task CA6-T238
/zoho-task 1013893000023483027
/zoho-task https://projects.zoho.com/portal/mtcmedialtd#milestone/1013893000022796035/1013893000022802009/tasklist-detail/1013893000023486013/task-detail/1013893000023483027
```

## What This Skill Does

This skill fetches the Zoho task and routes to different workflows based on the task status.

### Status: "To do"

The task needs implementation work. The skill will:

1. Read the task description to understand requirements
2. Read any existing comments for additional context
3. Explore the codebase to understand the current state
4. Create a todo list for implementation
5. Implement the required changes
6. Run tests to verify the implementation works
7. Create a feature branch following the naming convention: `feature/CA6-TXXX-description`
8. Create BitBucket PR (target: staging)
9. Update Zoho task status to "Open" and add comment with PR link

### Status: "Open" or "In Review"

The task should already be complete. The skill will:

1. Read the task description to understand what was implemented
2. Read existing comments for context on what was done
3. Write an E2E Playwright test to prove the feature works
4. Capture screenshots during the test (1440x900 viewport)
5. Upload screenshots to the Zoho task as attachments
6. Construct a Zoho comment with:
   - Summary of what was implemented (client-friendly language)
   - Inline embedded screenshots as evidence
   - Live production URLs where applicable
   - Steps the client can take to verify the feature

## Preview Before Updating Zoho

**Always preview changes before making any Zoho API call.** Before updating task status, adding comments, or editing comments, show the user exactly what will be sent and ask for confirmation.

Use the AskUserQuestion tool to present the preview:

```
AskUserQuestion:
  question: "Ready to update Zoho. Does this look correct?"
  header: "Confirm"
  options:
    - label: "Yes, update Zoho"
      description: "Proceed with the update as shown"
    - label: "No, let me edit"
      description: "I'll provide changes before updating"
```

**What to preview:**
- **Status changes**: Show the current status → new status
- **New comments**: Show the full comment content (formatted for readability)
- **Comment edits**: Show the original comment and the updated version

This gives the user a chance to catch issues before they're posted to the client-visible task.

## Instructions

When this skill is invoked:

### Step 0: Check Configuration

Before proceeding, check if the required configuration exists in the project's CLAUDE.md file. Look for these values:

- **Project ID** (Zoho)
- **Open Status ID** (Zoho)
- **Workspace** (BitBucket)
- **Repository** (BitBucket)
- **Production URL**

**If any configuration is missing**, use the AskUserQuestion tool to prompt for the missing values:

```
AskUserQuestion:
  question: "I need some configuration to work with Zoho tasks. What is your Zoho Project ID?"
  header: "Project ID"
  options:
    - label: "I'll provide it"
      description: "Enter your Zoho Project ID (found in the URL when viewing your project)"
```

Prompt for each missing value, then offer to save them to `.claude/CLAUDE.md`:

```
AskUserQuestion:
  question: "Would you like me to save this configuration to .claude/CLAUDE.md for future use?"
  header: "Save config"
  options:
    - label: "Yes, save it"
      description: "Add configuration to .claude/CLAUDE.md so you don't have to enter it again"
    - label: "No, just use it this time"
      description: "Use these values for this session only"
```

If the user chooses to save, create or update `.claude/CLAUDE.md` with the configuration block.

### Step 1: Parse the Task Reference

Extract the task identifier from the argument:

- **URL format**: Extract the task ID from the URL (the number after `task-detail/`)
- **Task ID format**: Use the numeric ID directly (16+ digits)
- **Prefix format**: Use `get_task_by_prefix` to find the task (e.g., `CA6-T176`)

### Step 2: Fetch the Task

Use the Zoho Projects MCP tools. **Always include the project_id** for faster, more reliable lookups:

```
# For prefix (e.g., CA6-T176) - ALWAYS include project_id
mcp__zoho-projects__get_task_by_prefix(prefix: "CA6-T176", project_id: "<PROJECT_ID>")

# For task ID
mcp__zoho-projects__get_task(project_id: "<PROJECT_ID>", task_id: "<task_id>")
```

**Note:** The project ID should be configured in your project's CLAUDE.md file.

Also fetch task comments for additional context:

```
mcp__zoho-projects__list_task_comments(project_id: "<PROJECT_ID>", task_id: "<task_id>")
```

### Step 3: Route Based on Status

Check the task's `status.name` field and route accordingly:

#### If status is "To do":

1. **Understand the task**: Read the description and comments carefully
2. **Explore the codebase**: Use the Task tool with `subagent_type=Explore` to understand:
   - Relevant existing code and patterns
   - Where changes need to be made
   - How similar features were implemented
3. **Plan the implementation**: Create a todo list with specific steps
4. **Create branch**: If not already on a feature branch, create one:
   ```
   git checkout -b feature/CA6-TXXX-short-description
   ```
5. **Implement**: Make the necessary code changes
6. **Test**: Run relevant tests to verify the implementation
7. **Commit and push**:
    ```bash
    git add -A
    git commit -m "feat(scope): description"
    git push -u origin feature/CA6-TXXX-short-description
    ```

8. **Create BitBucket PR** (target: `staging`):
    ```
    mcp__bitbucket__bb_post(
        path: "/repositories/<WORKSPACE>/<REPO_SLUG>/pullrequests",
        body: {
            "title": "<TASK_PREFIX>: Brief description",
            "source": {"branch": {"name": "feature/<TASK_PREFIX>-short-description"}},
            "destination": {"branch": {"name": "staging"}},
            "description": "## Summary\n- Change description\n\n## Zoho Task\n<TASK_PREFIX>"
        }
    )
    ```
    Capture the PR number and URL from the response.

9. **Update Zoho task status to "Open"**:
    The "Open" status ID should be configured in CLAUDE.md.

    **Preview first**: Show the user the status change (e.g., "To do" → "Open") and get confirmation before updating:
    ```
    mcp__zoho-projects__update_task(
        project_id: "<PROJECT_ID>",
        task_id: "<task_id>",
        status_id: "<OPEN_STATUS_ID>"
    )
    ```

10. **Add Zoho comment** with PR reference and feature summary:
    Always include a clear summary of what features the PR introduces, written in client-friendly language.

    **Preview first**: Show the user the full comment content (formatted for readability) and get confirmation before posting.
    ```
    mcp__zoho-projects__add_task_comment(
        project_id: "<PROJECT_ID>",
        task_id: "<task_id>",
        content: "Code ready for internal review:<ul><li>Branch: <code>feature/<TASK_PREFIX>-description</code></li><li>Pull Request: <a href=\"https://bitbucket.org/<WORKSPACE>/<REPO_SLUG>/pull-requests/XX\">PR #XX</a></li></ul><b>What this adds:</b><ul><li>Feature 1 in plain language (what users can now do)</li><li>Feature 2 in plain language</li><li>Any other key changes</li></ul>"
    )
    ```

    **Important**: The feature summary should:
    - Describe what users can now DO, not technical implementation details
    - Use client-friendly language (no jargon)
    - List all key features/changes introduced by the PR
    - Help reviewers understand the PR's purpose at a glance

#### If status is "Open" or "In Review":

1. **Understand what was done**: Read the description and comments
2. **Create screenshot test**: Write a Playwright E2E test that:
   - Navigates to the relevant page(s)
   - Captures screenshots at key steps
   - Uses 1440x900 viewport for clear images
   - Saves to `screenshots/<feature-name>/` with numbered filenames

   Example test structure:
   ```typescript
   import { test } from '@playwright/test'
   import fs from 'fs'
   import path from 'path'

   const screenshotDir = path.join(__dirname, '../../screenshots/<feature-name>')

   test.describe('Feature Name - Screenshots', () => {
       test.use({ viewport: { width: 1440, height: 900 } })

       test.beforeAll(async () => {
           if (!fs.existsSync(screenshotDir)) {
               fs.mkdirSync(screenshotDir, { recursive: true })
           }
       })

       test('capture feature screenshots', async ({ page }) => {
           await page.goto('/staff/...')
           await page.screenshot({
               path: path.join(screenshotDir, '01-step-name.png'),
               fullPage: false
           })
           // Additional steps...
       })
   })
   ```

3. **Run the test**: Execute the screenshot test
   ```bash
   CI=1 PLAYWRIGHT_BASE_URL=http://localhost:${APP_PORT} npx playwright test <test-file> --project=chromium
   ```

4. **Upload screenshots**: Use the Zoho MCP tool to upload each screenshot
   ```
   mcp__zoho-projects__upload_task_attachment(
       project_id: "<PROJECT_ID>",
       task_id: "<task_id>",
       file_path: "/absolute/path/to/screenshot.png"
   )
   ```
   Save the `third_party_file_id` and `x-cli-msg` from each response.

5. **Construct the Zoho screenshot comment**: Create an HTML comment following CLAUDE.md guidelines:
   - **Start with marker**: Begin with `[SCREENSHOTS]` so the comment is identifiable for later editing
   - Client-friendly language (no technical jargon)
   - Use HTML formatting (`<ul>`, `<li>`, `<b>` tags)
   - Embed screenshots inline using the `/image/` endpoint:
     ```
     https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}
     ```
   - Include production URLs where applicable (use `<PRODUCTION_URL>` from CLAUDE.md)
   - Keep the entire comment on ONE line (no line breaks except explicit `<br>`)
   - Include "How to verify" steps

   Example format:
   ```html
   [SCREENSHOTS] Feature evidence and verification:<ul><li>Feature description in plain language</li></ul><b>Screenshots:</b><br><img src="https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}" /><b>How to verify:</b><ul><li>Visit <a href="<PRODUCTION_URL>/staff/page">the page</a></li><li>Observe the feature working</li></ul>
   ```

6. **Post the screenshot comment**: Add the comment to the task

   **Preview first**: Show the user the full comment content (formatted for readability, with embedded image previews if possible) and get confirmation before posting.
   ```
   mcp__zoho-projects__add_task_comment(
       project_id: "<PROJECT_ID>",
       task_id: "<task_id>",
       content: "<html-comment>"
   )
   ```

   **Note**: The `[SCREENSHOTS]` marker makes this comment identifiable. To update screenshots later, find this comment using `list_task_comments` and use `edit_task_comment` with the comment ID.

### Step 4: Multiple PRs (When Applicable)

Create separate PRs when:
- **Feature + Evidence**: Main feature work merged earlier, now adding screenshot tests as evidence
- **Incremental delivery**: Large task broken into reviewable chunks
- **Fixes during review**: Additional changes needed after initial review

Reference all relevant PRs in the Zoho comment's internal section.

## Zoho Comment Templates

### For Client Review (Primary Section)

Write in plain English, focusing on what the user can now do:

```html
This feature is now complete and ready for your review.<ul><li>You can now [describe what they can do]</li><li>[Another user-facing benefit]</li></ul><b>Where to find it:</b><br>Visit <a href="<PRODUCTION_URL>/staff/page">the page name</a> to see this in action.<b>Screenshots:</b><img src="https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}" /><b>How to verify this is working:</b><ul><li>Go to [page name]</li><li>You should see [expected result]</li><li>Try [action] and confirm [outcome]</li></ul>
```

### Internal Reference Section (Optional)

If you need to include technical references for internal team members, add a clearly separated section:

```html
<hr><b>Internal Reference (for development team):</b><ul><li>Branch: <code>feature/<TASK_PREFIX>-description</code></li><li>Pull Request: <a href="https://bitbucket.org/<WORKSPACE>/<REPO_SLUG>/pull-requests/XX">PR #XX</a></li></ul>
```

### Complete Example

```html
This feature is now complete and ready for your review.<ul><li>Staff can now download session data as a spreadsheet file</li><li>The download includes all visible columns and respects any active filters</li></ul><b>Where to find it:</b><br>Visit <a href="<PRODUCTION_URL>/staff/sessions">the Sessions page</a> and look for the "Export" button in the top right.<b>Screenshots:</b><br><img src="https://previewengine-accl.zoho.com/image/WD/abc123?x-cli-msg=xyz" /><b>How to verify this is working:</b><ul><li>Go to the Sessions page</li><li>Click the "Export" button</li><li>A spreadsheet file should download to your computer</li><li>Open the file to confirm it contains the session data</li></ul><hr><b>Internal Reference:</b><ul><li>Branch: <code>feature/<TASK_PREFIX>-csv-export</code></li><li>PR: <a href="https://bitbucket.org/<WORKSPACE>/<REPO_SLUG>/pull-requests/45">PR #45</a></li></ul>
```

## Final Checklist

### For "To do" Tasks (Implementation)

Before completing the PR workflow:

- [ ] **Implementation complete**: All required functionality is working
- [ ] **Tests pass**: Unit/feature tests verify the implementation
- [ ] **PR created**: BitBucket PR targeting staging branch
- [ ] **Status updated**: Task status changed to "Open"
- [ ] **Zoho comment added**: PR link included for internal reference

### For "Open"/"In Review" Tasks (Documentation)

Before posting the Zoho screenshot comment, verify:

- [ ] **Marker included**: Comment starts with `[SCREENSHOTS]` for later identification
- [ ] **Language check**: No technical jargon - would a non-developer understand this?
- [ ] **Action-focused**: Describes what users can DO, not how it was built
- [ ] **Screenshots**: All images uploaded and embedded inline (using `/image/` not `/thumbnail/`)
- [ ] **Links**: Production URLs included for relevant pages
- [ ] **Verification steps**: Clear, actionable steps the client can follow
- [ ] **Single line**: Entire comment on ONE line (no unintended line breaks)
- [ ] **PR reference**: Included in internal section (if applicable)

## Important Notes

- **Preview before updating Zoho**: Always show the user what you're about to change before making any Zoho API calls (status updates, comments, edits). This gives them a chance to review and approve the changes.
- **Screenshots are gitignored**: Don't commit screenshot images to the repo (only the test files)
- **Single-line comments**: Zoho converts newlines to `<br>` tags - keep entire comment on one line
- **No emojis**: Don't use checkmarks (✅) or other emojis in Zoho comments
- **No dates**: Don't include "Status Update - January 2026" headers
- **Screenshot hosting in PRs**: Embed screenshots directly in BitBucket PR descriptions (BitBucket hosts them) - Zoho images are behind a login and won't display
- **Don't say "ready for review/testing"**: Only say this once changes are actually deployed and testable - code being in a PR doesn't mean it's ready for client review
- **Edit, don't add corrections**: If a comment needs correcting, use `edit_task_comment` to fix the original rather than adding follow-up comments

## Required Configuration

Add these values to your project's CLAUDE.md file:

```markdown
# Zoho Projects Configuration
- **Project ID**: `<your-project-id>`
- **Open Status ID**: `<your-open-status-id>`

# BitBucket Configuration
- **Workspace**: `<your-workspace>`
- **Repository**: `<your-repo-slug>`

# URLs
- **Production URL**: `<your-production-url>`
```

To find your status IDs, use:
```
mcp__zoho-projects__list_statuses(project_id: "<PROJECT_ID>")
```
