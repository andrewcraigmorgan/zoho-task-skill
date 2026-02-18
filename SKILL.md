---
name: zoho-task
description: Work on a Zoho Projects task based on its current status. Accepts URL, ID, or prefix (e.g., PRJ-T123).
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

- **Full URL**: `https://projects.zoho.com/portal/{portal}#milestone/.../task-detail/{task_id}`
- **Task ID**: The numeric task ID (16+ digits)
- **Task prefix/code**: e.g., `PRJ-T123`

Examples:

```
/zoho-task PRJ-T123
/zoho-task 1234567890123456789
/zoho-task https://projects.zoho.com/portal/{portal}#milestone/{ids}/task-detail/{task_id}
```

## What This Skill Does

This skill fetches the Zoho task and routes to different workflows based on the task status.

### Status Meanings

- **To do**: Task has not been started yet
- **Open**: Task has been picked up and work is in progress (PR created, awaiting merge/deployment)
- **In Review**: Feature is visible on production or testing environment and the client can verify it

**Important:** The skill should only change status from "To do" → "Open". The "In Review" status is reserved for when the client can actually see and test the feature - this transition happens manually after deployment.

### Status: "To do"

The task needs implementation work. The skill will:

1. Read the task description to understand requirements
2. Read any existing comments for additional context
3. Explore the codebase to understand the current state
4. Create a todo list for implementation
5. Implement the required changes
6. Run tests to verify the implementation works
7. Create a feature branch following the naming convention: `feature/{PREFIX}-TXXX-description`
8. Create BitBucket PR (target: staging)
9. Update Zoho task status to "Open" and add comment with PR link

### Status: "Open" or "In Review"

The task implementation is complete and visible (or will be visible soon). The skill will add documentation and screenshots but will **NOT change the status** (status transitions to "In Review" are done manually after deployment):

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
- **Prefix format**: Use `get_task_by_prefix` to find the task (e.g., `PRJ-T123`)

### Step 2: Fetch the Task

Use the Zoho Projects MCP tools. **Always include the project_id** for faster, more reliable lookups:

```
# For prefix (e.g., PRJ-T123) - ALWAYS include project_id
mcp__zoho-projects__get_task_by_prefix(prefix: "PRJ-T123", project_id: "<project_id>")

# For task ID
mcp__zoho-projects__get_task(project_id: "<project_id>", task_id: "<task_id>")
```

**Note:** Get the default project ID from the project's CLAUDE.md "Zoho Projects Configuration" section.

Also fetch task comments for additional context:

```
mcp__zoho-projects__list_task_comments(project_id: "<project_id>", task_id: "<task_id>")
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
   git checkout -b feature/{PREFIX}-TXXX-short-description
   ```
5. **Implement**: Make the necessary code changes
6. **Test**: Run relevant tests to verify the implementation
7. **Commit and push**:
    ```bash
    git add -A
    git commit -m "feat(scope): description"
    git push -u origin feature/{PREFIX}-TXXX-short-description
    ```

8. **Create BitBucket PR** (target: `staging`, draft mode, with default reviewers):

    First, fetch the repository's default reviewers WITH display names:
    ```
    mcp__bitbucket__bb_get(
        path: "/repositories/{workspace}/{repo-slug}/default-reviewers",
        jq: "values[*].{uuid: uuid, display_name: display_name, nickname: nickname}"
    )
    ```

    **CRITICAL: Filter out the author from reviewers.** Bitbucket rejects PRs where the author is listed as a reviewer. Check the "Known Team Members" table in the bitbucket-helpers skill or use `git config user.email` to identify the author, then exclude their UUID.

    Then create the PR with only non-author reviewers:
    ```
    mcp__bitbucket__bb_post(
        path: "/repositories/{workspace}/{repo-slug}/pullrequests",
        body: {
            "title": "XXX-TXXX: Brief description",
            "source": {"branch": {"name": "feature/XXX-TXXX-short-description"}},
            "destination": {"branch": {"name": "staging"}},
            "description": "## Summary\n- Change description\n\n## Zoho Task\nXXX-TXXX",
            "draft": true,
            "reviewers": [{"uuid": "{non-author-uuid}"}]  // Exclude author's UUID!
        }
    )
    ```
    Get the workspace/repo-slug from the git remote or project's CLAUDE.md.
    Capture the PR number and URL from the response.
    **Note:** PRs are created in draft mode. Only include reviewers who are NOT the author.

9. **Update Zoho task status to "Open"**:
    Get the "Open" status ID from the project's CLAUDE.md "Zoho Projects Configuration" section.
    If not found, check the "Project Status IDs Reference" section below or ask the user.
    ```
    mcp__zoho-projects__update_task(
        project_id: "<project_id_from_claude_md>",
        task_id: "<task_id>",
        status_id: "<open_status_id_from_claude_md>"
    )
    ```

10. **Preview and add Zoho comment** with PR reference, feature summary, and verification steps:

    **First, show the user a FORMATTED preview using markdown (not raw HTML) and wait for approval.** Example:

    > ---
    >
    > Code ready for internal review:
    >
    > - Branch: `feature/{PREFIX}-TXXX-description`
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
        project_id: "<project_id>",
        task_id: "<task_id>",
        content: "Code ready for internal review:<ul><li>Branch: <code>feature/{PREFIX}-TXXX-description</code></li><li>Pull Request: <a href=\"https://bitbucket.org/.../pull-requests/XX\">PR #XX</a></li></ul><b>What this adds:</b><ul><li>Feature 1 in plain language (what users can now do)</li><li>Feature 2 in plain language</li></ul><b>How to verify (once deployed):</b><ol><li>Go to <a href=\"{STAGING_URL}/relevant-page\">the relevant page</a></li><li>Perform [specific action]</li><li>You should see [expected result]</li></ol>"
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

2. **Check for existing screenshot evidence comment FIRST**:

   Before doing any work, check if a screenshot evidence comment already exists:

   ```
   mcp__zoho-projects__list_task_comments(project_id: "...", task_id: "...")
   ```

   Search each comment's `comment` field for the string `<!-- screenshots-evidence -->`.

   **If an existing comment is found**, ask the user what they want to do:

   > "I found an existing screenshot evidence comment on this task.
   >
   > What would you like to do?
   > 1. **Update screenshots** - Capture new screenshots and replace the existing ones
   > 2. **Edit comment text** - Modify the wording without changing screenshots
   > 3. **Skip** - No changes needed to the evidence comment
   >
   > (Or describe what specific changes you'd like to make)"

   **Wait for the user's response before proceeding.** Based on their answer:
   - If "update screenshots" or similar → Continue to step 3 (create screenshot test)
   - If "edit comment text" → Show the current comment content and ask what to change, then update
   - If "skip" → Do not modify the comment; ask if there's anything else to do for this task

   **If no existing comment is found**, proceed to step 3 to create screenshots and a new comment.

3. **Ensure latest staging changes are merged**:

   Before running any screenshot tests, ensure your branch has the latest staging changes to capture current styles:

   ```bash
   # Fetch latest and merge staging (don't checkout - may be used by another worktree)
   git fetch origin
   git merge origin/staging --no-edit
   ```

   If using Docker, rebuild the frontend after merging:
   ```bash
   docker compose exec app npm run build
   ```

   If running locally:
   ```bash
   npm run build
   ```

   **Why this matters:** Screenshot tests capture the UI as it appears. If your branch is behind staging, the screenshots will show outdated styles or missing features, requiring them to be recaptured later.

4. **Create screenshot test**: Write a Playwright E2E test that:
   - Navigates to the relevant page(s)
   - Captures screenshots at key steps
   - Uses the shared viewport config (check project's CLAUDE.md for default size)
   - Saves to `screenshots/<feature-name>/` with numbered filenames

   Example test structure:
   ```typescript
   import { test } from '@playwright/test'
   import fs from 'fs'
   import path from 'path'
   import { screenshotViewport } from '../screenshot.config'

   const screenshotDir = path.join(__dirname, '../../screenshots/<feature-name>')

   test.describe('Feature Name - Screenshots', () => {
       test.use({ viewport: screenshotViewport })

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

5. **Run the test**: Execute the screenshot test
   ```bash
   CI=1 PLAYWRIGHT_BASE_URL=http://localhost:${APP_PORT} npx playwright test <test-file> --project=chromium
   ```

6. **Upload screenshots**: Use the Zoho MCP tool to upload each screenshot
   ```
   mcp__zoho-projects__upload_task_attachment(
       project_id: "<project_id>",
       task_id: "<task_id>",
       file_path: "/absolute/path/to/screenshot.png"
   )
   ```
   Save the `third_party_file_id` and `x-cli-msg` from each response.

7. **Construct the Zoho screenshot comment**: Create an HTML comment following CLAUDE.md guidelines:
   - **No redundant titles**: Do NOT start with "[SCREENSHOTS]", "Feature verification", or similar meta-titles. Jump straight into the content.
   - Client-friendly language (no technical jargon)
   - Use HTML formatting (`<ul>`, `<li>`, `<b>` tags)
   - Embed screenshots inline using the `/image/` endpoint:
     ```
     https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}
     ```
   - Include environment URLs where applicable (from project's CLAUDE.md)
   - Keep the entire comment on ONE line (no line breaks except explicit `<br>`)
   - Include "How to verify" steps
   - **End with hidden marker**: Always append `<!-- screenshots-evidence -->` at the end for future identification (this is invisible to readers but allows the system to find and update the comment later)

   Example format:
   ```html
   The [feature name] is now complete. Here's what you can do:<ul><li>Key capability 1</li><li>Key capability 2</li></ul><b>1. [Screenshot description]:</b><br/><img src="https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}" /><br/><br/><b>How to verify:</b><ol><li>Visit <a href="{PRODUCTION_URL}/staff/page">the page</a></li><li>Perform [specific action]</li><li>Observe [expected result]</li></ol><!-- screenshots-evidence -->
   ```

   **Note:** Replace `{PRODUCTION_URL}` with the production URL from the project's CLAUDE.md.

8. **Preview and post/update the screenshot comment**:

   **First, show the user a FORMATTED preview using markdown (not raw HTML) so they can easily read and review the wording.** Wrap in horizontal rules (`---`) to visually separate it.

   **Only after user approval**, add or update the comment:

   If **creating new** (no existing screenshot comment found):
   ```
   mcp__zoho-projects__add_task_comment(
       project_id: "<project_id>",
       task_id: "<task_id>",
       content: "<html-comment>"
   )
   ```

   If **updating existing** (screenshot comment already exists):
   ```
   mcp__zoho-projects__edit_task_comment(
       project_id: "<project_id>",
       task_id: "<task_id>",
       comment_id: "<existing_comment_id>",
       content: "<html-comment>"
   )
   ```

   **Note**: The hidden `<!-- screenshots-evidence -->` marker makes this comment identifiable for future updates without cluttering the visible content.

### Step 4: Multiple PRs (When Applicable)

Create separate PRs when:
- **Feature + Evidence**: Main feature work merged earlier, now adding screenshot tests as evidence
- **Incremental delivery**: Large task broken into reviewable chunks
- **Fixes during review**: Additional changes needed after initial review

Reference all relevant PRs in the Zoho comment's internal section.

## Zoho Comment Templates

**IMPORTANT: Be concise.**
- No meta-titles ("[SCREENSHOTS]", "Feature verification:", "Status update:")
- No redundant headers ("Screenshots:" before images - they're obviously screenshots)
- No filler phrases ("This feature is now complete and ready for your review")
- Jump straight to what matters: what changed, evidence, how to verify

### For Client Review (Primary Section)

Start with what the user can now do, show the evidence, explain how to verify:

```html
[Brief description of what's new]:<ul><li>[Key capability 1]</li><li>[Key capability 2]</li></ul><b>1. [What this screenshot shows]:</b><br/><img src="..." /><br/><br/><b>2. [What this screenshot shows]:</b><br/><img src="..." /><br/><br/><b>How to verify:</b><ol><li>[Action step]</li><li>[Expected result]</li></ol><!-- screenshots-evidence -->
```

**Note:**
- Use `<ol>` for verification steps (numbered: 1, 2, 3...)
- Replace URLs with values from project's CLAUDE.md "Environment URLs" section

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
This feature is now complete and ready for your review.<ul><li>Staff can now download session data as a spreadsheet file</li><li>The download includes all visible columns and respects any active filters</li></ul><b>Where to find it:</b><br>Visit <a href="{PRODUCTION_URL}/staff/sessions">the Sessions page</a> and look for the "Export" button in the top right.<b>Screenshots:</b><br><img src="https://previewengine-accl.zoho.com/image/WD/abc123?x-cli-msg=xyz" /><b>How to verify this is working:</b><ol><li>Go to the Sessions page</li><li>Click the "Export" button</li><li>A spreadsheet file should download to your computer</li><li>Open the file to confirm it contains the session data</li></ol><hr><b>Internal Reference:</b><ul><li>Branch: <code>feature/XXX-TXXX-csv-export</code></li><li>PR: <a href="{BITBUCKET_PR_URL}">PR #45</a></li></ul><!-- screenshots-evidence -->
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
- [ ] **Status updated**: Task status changed to "Open" (use status ID from project's CLAUDE.md)
- [ ] **Comment preview shown**: User has seen and approved the Zoho comment content
- [ ] **Zoho comment added** with ALL of the following:
  - [ ] PR link for internal reference
  - [ ] Feature summary in plain language
  - [ ] **Verification steps**: Numbered steps to test on staging environment
  - [ ] **Staging URL**: Link to the relevant page using the staging URL from project's CLAUDE.md
  - [ ] **Expected outcomes**: What the reviewer should see

### For "Open"/"In Review" Tasks (Documentation)

Before doing any screenshot work:

- [ ] **Check for existing FIRST**: Search comments for `<!-- screenshots-evidence -->` marker
- [ ] **Ask user if existing found**: If comment exists, ask what to do (update screenshots, edit text, or skip)
- [ ] **Wait for user response**: Do NOT proceed with screenshot capture until user confirms what they want

If proceeding with screenshot work:

- [ ] **Preview approved**: User has seen and approved the comment content
- [ ] **Hidden marker included**: Comment ends with `<!-- screenshots-evidence -->` for future updates
- [ ] **Language check**: No technical jargon - would a non-developer understand this?
- [ ] **Action-focused**: Describes what users can DO, not how it was built
- [ ] **Screenshots**: All images uploaded and embedded inline (using `/image/` not `/thumbnail/`)
- [ ] **Links**: Production URLs included for relevant pages
- [ ] **Verification steps**: Clear, actionable steps the client can follow
- [ ] **Single line**: Entire comment on ONE line (no unintended line breaks)
- [ ] **Update vs Create**: If existing comment found, use `edit_task_comment`; otherwise use `add_task_comment`

## Clarification Workflow

When a task requires clarification before implementation can begin:

1. **Identify the need**: If the task description is ambiguous, has multiple valid approaches, or requires client input, draft a clarification question.

2. **Draft the comment**: Create a client-friendly comment presenting the options clearly (see example below).

3. **Get user approval**: Show the preview and wait for explicit approval before posting.

4. **Post comment and update status**: After posting, update the task status to "Awaiting Approval" (or equivalent) to indicate the task is blocked pending client input.

5. **Wait for response**: Do not proceed with implementation until the client responds.

### Clarification Comment Template

```html
<b>Clarification needed on approach:</b><br><br>We have two options for this feature:<br><br><b>Option A - [Name] (Recommended)</b><ul><li>[Benefit 1]</li><li>[Benefit 2]</li></ul><b>Option B - [Name]</b><ul><li>[Benefit 1]</li><li>[Benefit 2]</li></ul>Which approach would work better for your needs?
```

### Status Update for Clarification

After posting a clarification comment, update the task status:

```
mcp__zoho-projects__update_task(
    project_id: "<project_id>",
    task_id: "<task_id>",
    status_id: "<awaiting_approval_status_id>"
)
```

## Project Status IDs Reference

**IMPORTANT: Always check the project's CLAUDE.md first for Zoho configuration.**

Status IDs vary by project. The correct approach is:

### Step 1: Check Project CLAUDE.md

Look for a "Zoho Projects Configuration" section in the project's CLAUDE.md file. This section should contain:
- Project ID
- Task Prefix (e.g., PRJ)
- Status IDs table with Open, To Do, Closed, etc.

**If the project CLAUDE.md has Zoho configuration, use those status IDs.**

### Step 2: If No Configuration Found

If the project's CLAUDE.md does not have a "Zoho Projects Configuration" section:

1. **Ask the user** before proceeding:
   > "I couldn't find Zoho status IDs in this project's CLAUDE.md. Would you like me to:
   > 1. Look up the status IDs from the Zoho API and add them to CLAUDE.md?
   > 2. Use the fallback reference below?"

2. If the user chooses option 1, fetch a task with "Open" status from the project:
   ```
   mcp__zoho-projects__search(search_term: "Open", module: "tasks", project_id: "<project_id>")
   ```
   Then add the configuration to the project's CLAUDE.md.

### Finding Status IDs

If you need a status ID not listed:
1. Search for tasks in that status: `mcp__zoho-projects__search(search_term: "status_name", module: "tasks", project_id: "...")`
2. Look at the `status.id` field in any task response
3. **Add the status ID to the project's CLAUDE.md** (preferred) or this fallback reference

## Zoho Task URL Format

When providing links to Zoho tasks, use this URL format:

```
https://projects.zoho.com/portal/{portal}#milestone/{milestone_id}/{project_id}/tasklist-detail/{tasklist_id}/task-detail/{task_id}
```

To construct the URL, extract these IDs from the task response:
- Portal name from the project's CLAUDE.md or Zoho configuration
- `milestone.id` → `{milestone_id}`
- `project.id` → `{project_id}`
- `tasklist.id` → `{tasklist_id}`
- `id` → `{task_id}`

**Note:** The simplified format (`#taskdetail/{project_id}/{task_id}`) does NOT work reliably. Always use the full milestone/tasklist format above.

## Screenshot Viewport Configuration

Screenshot tests should use a shared viewport config that can be customized per-project.

### Default Viewport

The default viewport for new projects is **Desktop (1440x900)**. Individual projects can override this in their `screenshot.config.ts`.

### Common Viewport Sizes

| Device | Dimensions | Use Case |
|--------|------------|----------|
| Desktop | 1440x900 | Staff dashboard, admin screens (default) |
| iPad Portrait | 768x1024 | Client-facing forms, mobile-first UI |
| iPad Landscape | 1024x768 | Tablet landscape mode |
| Mobile | 390x844 | iPhone-style mobile testing |

### Project Configuration

Check the project's CLAUDE.md for a "Screenshot Configuration" section specifying the preferred viewport. The project's `tests/e2e/screenshot.config.ts` defines the actual default used by tests.

## Important Notes

- **Project ID**: Check project's CLAUDE.md "Zoho Projects Configuration" section
- **Open Status ID**: Get from project's CLAUDE.md "Zoho Projects Configuration" section (varies by project)
- **Never set "In Review"**: The skill should never change a task to "In Review" status - that is reserved for when the client can see the feature on production/testing and is done manually
- **Awaiting Approval**: Use when task is blocked pending client clarification
- **Environment URLs**: Defined in project's CLAUDE.md under "Environment URLs" section
  - Use **Staging URL** for verification steps when PR is merged to staging
  - Use **Production URL** only after deployment to production
- **BitBucket Repo**: Check project's CLAUDE.md or git remote for workspace/repo-slug
- **Screenshots are gitignored**: Don't commit screenshot images to the repo (only the test files)
- **Single-line comments**: Zoho converts newlines to `<br>` tags - keep entire comment on one line
- **No emojis**: Don't use checkmarks (✅) or other emojis in Zoho comments
- **No dates**: Don't include "Status Update - January 2026" headers
- **Screenshot hosting**: Upload to Zoho first to get URLs, then embed in BitBucket PR description
