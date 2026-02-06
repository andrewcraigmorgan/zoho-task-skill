---
name: zoho-task
description: Work on a Zoho Projects task based on its current status. Accepts URL, ID, or prefix (e.g., CA6-T238).
---

# Zoho Task Skill

Work on a Zoho Projects task based on its current status.

## CRITICAL: Zoho Comment Approval Gate

**NEVER call `add_task_comment` or `edit_task_comment` without EXPLICIT user approval.**

### Mandatory Approval Process

Before ANY Zoho comment action, you MUST complete this checklist:

1. **Show a FORMATTED preview** of the comment using markdown (not raw HTML) so the user can read it easily. Use markdown bullet points, bold, numbered lists, and links to represent the HTML structure. Wrap the preview in a horizontal rule block (`---`) to visually separate it.

2. **Ask explicitly**: "Should I post this comment to Zoho? (yes/no)"

3. **STOP and WAIT** for the user to respond with explicit approval (e.g., "yes", "post it", "approved")

4. **If the user says anything OTHER than clear approval** (e.g., asks a question, points out an issue, says "there's no content"), do NOT post. Clarify first.

### What Counts as Approval

✓ "yes"
✓ "post it"
✓ "approved"
✓ "go ahead"
✓ "looks good, post it"

### What Does NOT Count as Approval

✗ No response yet (WAIT)
✗ User asks a question (ANSWER FIRST)
✗ User points out an issue (FIX FIRST)
✗ User says anything ambiguous (CLARIFY FIRST)
✗ Silence or moving to next topic (DO NOT ASSUME APPROVAL)

### Content Validation

Before posting any comment, validate the content:

1. **Non-empty**: Never post empty or whitespace-only comments. If the content is blank, stop and ask the user what should be included.
2. **Minimum length**: Comments must be at least 10 characters of meaningful content. Short or placeholder text (e.g., "update", "done") is not acceptable for client-facing communication.
3. **No placeholders**: Ensure all template placeholders (e.g., `{STAGING_URL}`, `{file_id}`) have been replaced with actual values before posting.

If any validation fails, show the user what's wrong and ask them to provide or confirm the content before proceeding.

### Consequences of Violation

Posting without approval wastes client-facing communication and erodes trust. If you violate this rule, immediately acknowledge the mistake and ask the user what corrective action to take.

This applies to ALL Zoho comments - PR updates, screenshot comments, status updates, etc.

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

## Preview/Staging Environment

**All Zoho comments MUST include verification steps using the staging environment.**

The staging and production URLs are defined in the **project's CLAUDE.md** under "Environment URLs". Look for:
- **Staging URL** - For testing PRs before production (use in verification steps)
- **Production URL** - Live environment (only reference after deployment)

When writing verification steps:
- Always use the **staging URL** for tasks in "To do" → "Open" workflow (PR merged to staging)
- Use the **production URL** only for tasks in "Open"/"In Review" that have been deployed to production
- Provide specific page paths (e.g., `/staff/settings`, `/staff/manage-team`)
- Include login instructions if relevant (e.g., "Log in as a superuser")

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

## Instructions

When this skill is invoked:

### Step 1: Parse the Task Reference

Extract the task identifier from the argument:

- **URL format**: Extract the task ID from the URL (the number after `task-detail/`)
- **Task ID format**: Use the numeric ID directly (16+ digits)
- **Prefix format**: Use `get_task_by_prefix` to find the task (e.g., `CA6-T176`)

### Step 2: Fetch the Task

Use the Zoho Projects MCP tools. **Always include the project_id** for faster, more reliable lookups:

```
# For prefix (e.g., CA6-T176) - ALWAYS include project_id
mcp__zoho-projects__get_task_by_prefix(prefix: "CA6-T176", project_id: "1013893000022796035")

# For task ID
mcp__zoho-projects__get_task(project_id: "1013893000022796035", task_id: "<task_id>")
```

**Note:** The default project ID is `1013893000022796035` (CAHER project). Always use this when searching by prefix.

Also fetch task comments for additional context:

```
mcp__zoho-projects__list_task_comments(project_id: "1013893000022796035", task_id: "<task_id>")
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

8. **Create BitBucket PR** (target: `staging`, draft mode, with default reviewers):

    First, fetch the repository's default reviewers:
    ```
    mcp__bitbucket__bb_get(
        path: "/repositories/{workspace}/{repo-slug}/default-reviewers",
        jq: "values[*].{uuid: uuid}"
    )
    ```

    Then create the PR, including the default reviewers:
    ```
    mcp__bitbucket__bb_post(
        path: "/repositories/{workspace}/{repo-slug}/pullrequests",
        body: {
            "title": "XXX-TXXX: Brief description",
            "source": {"branch": {"name": "feature/XXX-TXXX-short-description"}},
            "destination": {"branch": {"name": "staging"}},
            "description": "## Summary\n- Change description\n\n## Zoho Task\nXXX-TXXX",
            "draft": true,
            "reviewers": [{"uuid": "{user-uuid-1}"}, {"uuid": "{user-uuid-2}"}]
        }
    )
    ```
    Get the workspace/repo-slug from the git remote or project's CLAUDE.md.
    Capture the PR number and URL from the response.
    **Note:** PRs are created in draft mode with all default reviewers added.

9. **Update Zoho task status to "Open"**:
    The "Open" status ID is `1013893000001076068`. Update the task:
    ```
    mcp__zoho-projects__update_task(
        project_id: "1013893000022796035",
        task_id: "<task_id>",
        status_id: "1013893000001076068"
    )
    ```

10. **Preview and add Zoho comment** with PR reference, feature summary, and verification steps:

    **First, show the user a FORMATTED preview using markdown (not raw HTML) and wait for approval.** Example:

    > ---
    >
    > Code ready for internal review:
    >
    > - Branch: `feature/CA6-TXXX-description`
    > - Pull Request: [PR #XX](https://bitbucket.org/...)
    >
    > **What this adds:**
    >
    > - Feature 1 in plain language
    > - Feature 2 in plain language
    >
    > **How to verify (once deployed):**
    >
    > 1. Go to [the relevant page](https://...)
    > 2. Perform [specific action]
    > 3. You should see [expected result]
    >
    > ---
    >
    > Should I post this to Zoho?

    **Only after user approval**, post the comment:
    ```
    mcp__zoho-projects__add_task_comment(
        project_id: "1013893000022796035",
        task_id: "<task_id>",
        content: "Code ready for internal review:<ul><li>Branch: <code>feature/CA6-TXXX-description</code></li><li>Pull Request: <a href=\"https://bitbucket.org/.../pull-requests/XX\">PR #XX</a></li></ul><b>What this adds:</b><ul><li>Feature 1 in plain language (what users can now do)</li><li>Feature 2 in plain language</li></ul><b>How to verify (once deployed):</b><ol><li>Go to <a href=\"{STAGING_URL}/relevant-page\">the relevant page</a></li><li>Perform [specific action]</li><li>You should see [expected result]</li></ol>"
    )
    ```

    **Important**: The comment MUST include:
    - **Feature summary**: Describe what users can now DO, not technical implementation details
    - **Verification steps**: Clear, numbered steps to test the feature on the staging/preview environment
    - **Staging URL**: Use the staging URL from the project's CLAUDE.md "Environment URLs" section
    - **Expected outcomes**: What the reviewer should see when following the steps
    - Use client-friendly language (no jargon)

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
       project_id: "1013893000022796035",
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
   - Include environment URLs where applicable (from project's CLAUDE.md)
   - Keep the entire comment on ONE line (no line breaks except explicit `<br>`)
   - Include "How to verify" steps

   Example format:
   ```html
   [SCREENSHOTS] Feature evidence and verification:<ul><li>Feature description in plain language</li></ul><b>Screenshots:</b><br><img src="https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}" /><b>How to verify:</b><ol><li>Visit <a href="{PRODUCTION_URL}/staff/page">the page</a></li><li>Perform [specific action]</li><li>Observe [expected result]</li></ol>
   ```

   **Note:** Replace `{PRODUCTION_URL}` with the production URL from the project's CLAUDE.md.

6. **Preview and post the screenshot comment**:

   **First, show the user a FORMATTED preview using markdown (not raw HTML) so they can easily read and review the wording.** Wrap in horizontal rules (`---`) to visually separate it.

   **Only after user approval**, add the comment to the task:
   ```
   mcp__zoho-projects__add_task_comment(
       project_id: "1013893000022796035",
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
This feature is now complete and ready for your review.<ul><li>You can now [describe what they can do]</li><li>[Another user-facing benefit]</li></ul><b>Where to find it:</b><br>Visit <a href="{PRODUCTION_URL}/staff/page">the page name</a> to see this in action.<b>Screenshots:</b><img src="https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}" /><b>How to verify this is working:</b><ol><li>Go to [page name]</li><li>You should see [expected result]</li><li>Try [action] and confirm [outcome]</li></ol>
```

**Note:**
- Use `<ol>` (ordered list) for verification steps so they appear as numbered steps (1, 2, 3...).
- Replace `{PRODUCTION_URL}` with the URL from the project's CLAUDE.md "Environment URLs" section.

### Internal Reference Section (Optional)

If you need to include technical references for internal team members, add a clearly separated section:

```html
<hr><b>Internal Reference (for development team):</b><ul><li>Branch: <code>feature/XXX-TXXX-description</code></li><li>Pull Request: <a href="{BITBUCKET_PR_URL}">PR #XX</a></li></ul>
```

### Complete Example (For "To do" → "Open" with staging URL)

```html
Code ready for internal review:<ul><li>Branch: <code>feature/XXX-TXXX-csv-export</code></li><li>Pull Request: <a href="{BITBUCKET_PR_URL}">PR #45</a></li></ul><b>What this adds:</b><ul><li>Staff can now download session data as a spreadsheet file</li><li>The download includes all visible columns and respects any active filters</li></ul><b>How to verify (once deployed):</b><ol><li>Go to <a href="{STAGING_URL}/staff/sessions">the Sessions page</a></li><li>Click the "Export" button in the top right</li><li>A spreadsheet file should download to your computer</li><li>Open the file to confirm it contains the session data</li></ol>
```

### Complete Example (For "Open"/"In Review" with production URL and screenshots)

```html
This feature is now complete and ready for your review.<ul><li>Staff can now download session data as a spreadsheet file</li><li>The download includes all visible columns and respects any active filters</li></ul><b>Where to find it:</b><br>Visit <a href="{PRODUCTION_URL}/staff/sessions">the Sessions page</a> and look for the "Export" button in the top right.<b>Screenshots:</b><br><img src="https://previewengine-accl.zoho.com/image/WD/abc123?x-cli-msg=xyz" /><b>How to verify this is working:</b><ol><li>Go to the Sessions page</li><li>Click the "Export" button</li><li>A spreadsheet file should download to your computer</li><li>Open the file to confirm it contains the session data</li></ol><hr><b>Internal Reference:</b><ul><li>Branch: <code>feature/XXX-TXXX-csv-export</code></li><li>PR: <a href="{BITBUCKET_PR_URL}">PR #45</a></li></ul>
```

**Note:** Replace placeholders with actual values:
- `{STAGING_URL}` / `{PRODUCTION_URL}` - From project's CLAUDE.md "Environment URLs" section
- `{BITBUCKET_PR_URL}` - The actual PR URL from BitBucket

## Final Checklist

### For "To do" Tasks (Implementation)

Before completing the PR workflow:

- [ ] **Implementation complete**: All required functionality is working
- [ ] **Tests pass**: Unit/feature tests verify the implementation
- [ ] **PR created**: BitBucket PR targeting staging branch
- [ ] **Status updated**: Task status changed to "Open" (ID: `1013893000001076068`)
- [ ] **Comment preview shown**: User has seen and approved the Zoho comment content
- [ ] **Zoho comment added** with ALL of the following:
  - [ ] PR link for internal reference
  - [ ] Feature summary in plain language
  - [ ] **Verification steps**: Numbered steps to test on staging environment
  - [ ] **Staging URL**: Link to the relevant page using the staging URL from project's CLAUDE.md
  - [ ] **Expected outcomes**: What the reviewer should see

### For "Open"/"In Review" Tasks (Documentation)

Before posting the Zoho screenshot comment, verify:

- [ ] **Preview approved**: User has seen and approved the comment content
- [ ] **Marker included**: Comment starts with `[SCREENSHOTS]` for later identification
- [ ] **Language check**: No technical jargon - would a non-developer understand this?
- [ ] **Action-focused**: Describes what users can DO, not how it was built
- [ ] **Screenshots**: All images uploaded and embedded inline (using `/image/` not `/thumbnail/`)
- [ ] **Links**: Production URLs included for relevant pages
- [ ] **Verification steps**: Clear, actionable steps the client can follow
- [ ] **Single line**: Entire comment on ONE line (no unintended line breaks)
- [ ] **PR reference**: Included in internal section (if applicable)

## Important Notes

- **Project ID**: Check project's CLAUDE.md or use default `1013893000022796035`
- **Open Status ID**: Use `1013893000001076068` to set task status to "Open"
- **Environment URLs**: Defined in project's CLAUDE.md under "Environment URLs" section
  - Use **Staging URL** for verification steps when PR is merged to staging
  - Use **Production URL** only after deployment to production
- **BitBucket Repo**: Check project's CLAUDE.md or git remote for workspace/repo-slug
- **Screenshots are gitignored**: Don't commit screenshot images to the repo (only the test files)
- **Single-line comments**: Zoho converts newlines to `<br>` tags - keep entire comment on one line
- **No emojis**: Don't use checkmarks (✅) or other emojis in Zoho comments
- **No dates**: Don't include "Status Update - January 2026" headers
- **Screenshot hosting**: Upload to Zoho first to get URLs, then embed in BitBucket PR description
