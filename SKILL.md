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

## MANDATORY: Clickable Preview Links

**Every Zoho comment MUST include a clickable link to the working feature.**

The client should be able to click directly from the Zoho comment to see the feature in action. This is non-negotiable.

### Where to Find the Preview URL

**Each project defines its own preview URL in the project's CLAUDE.md** under "Environment URLs". This is the URL where clients verify features.

Look for a section like:
```
## Environment URLs
| Environment | URL | Description |
| Production | https://app.example.com | Live environment |
```

Or explicit guidance like:
```
**Preview URL for Zoho comments:** Use **Production** (`https://app.example.com`) for client verification steps.
```

### CRITICAL: Always Use the Project's Preview URL

**For ALL Zoho comments with verification steps, use the preview URL defined in the project's CLAUDE.md.**

- The preview URL varies by project - check CLAUDE.md each time
- Clients click these links to verify features work
- Using the wrong URL means the client cannot verify the feature

**Before posting ANY Zoho comment, verify:**
- [ ] You have checked the project's CLAUDE.md for the correct preview URL
- [ ] All `<a href="...">` links use the URL specified for client verification
- [ ] No internal-only URLs (staging, localhost, dev servers) appear in the comment

### Link Requirements

Every comment with verification steps MUST include:

1. **At least one clickable link** to where the feature can be seen/tested
2. **Specific page paths** (e.g., `/staff/settings`, `/staff/manage-team`) - not just the base URL
3. **Descriptive link text** - use "the Sessions page" not "click here" or raw URLs
4. **Login instructions** if relevant (e.g., "Log in as a superuser")

### Examples

**Good** - Clickable link with specific path:
```html
<ol><li>Visit <a href="https://hercitizenadvice.mtcserver26.com/staff/sessions">the Sessions page</a></li>...</ol>
```

**Bad** - No clickable link:
```html
<ol><li>Go to the Sessions page</li>...</ol>
```

**Bad** - Only base URL without specific path:
```html
<ol><li>Visit <a href="https://hercitizenadvice.mtcserver26.com">the site</a> and navigate to Sessions</li>...</ol>
```

### Validation Before Posting

Before posting ANY comment with verification steps, check:
- [ ] Is there at least one `<a href="...">` link in the verification steps?
- [ ] Does the link include the full path to the relevant page?
- [ ] Is the URL from the project's CLAUDE.md (the preview URL where clients verify features)?

If any check fails, add the missing link before showing the preview to the user.

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
- **Awaiting Approval**: Task is in the scoping phase - requirements need clarification before implementation
- **Need more information**: Task is blocked waiting for client input (similar to Awaiting Approval)
- **Open**: Task has been picked up and work is in progress (PR created, awaiting merge/deployment)
- **In Review**: Feature is visible on production or testing environment and the client can verify it

**Important:** The skill should only change status from "To do" → "Open". The "In Review" status is reserved for when the client can actually see and test the feature - this transition happens manually after deployment. Tasks in "Awaiting Approval" or "Need more information" stay in that status until the client responds.

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

### Status: "Awaiting Approval" or "Need more information"

The task is blocked pending client input - either in the scoping phase or waiting for additional information. The skill will:

1. Read the task description to understand the initial request
2. Read any existing comments for prior context or partial answers
3. Explore the codebase to understand what's involved and identify technical considerations
4. Identify gaps, ambiguities, or decision points that need client input
5. Generate targeted clarification questions (5-8 questions max)
6. Preview questions in client-friendly language for user approval
7. Post approved questions as a Zoho comment
8. Keep status unchanged (waiting for client response)

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
        content: "Code ready for internal review:<ul><li>Branch: <code>feature/{PREFIX}-TXXX-description</code></li><li>Pull Request: <a href=\"https://bitbucket.org/.../pull-requests/XX\">PR #XX</a></li></ul><b>What this adds:</b><ul><li>Feature 1 in plain language (what users can now do)</li><li>Feature 2 in plain language</li></ul><b>How to verify (once deployed):</b><ol><li>Go to <a href=\"{PREVIEW_URL}/relevant-page\">the relevant page</a></li><li>Perform [specific action]</li><li>You should see [expected result]</li></ol>"
    )
    ```

    **Important**: The comment MUST include:
    - **Feature summary**: Describe what users can now DO, not technical implementation details
    - **Verification steps**: Clear, numbered steps to test the feature
    - **Preview URL**: Use the preview URL from the project's CLAUDE.md "Environment URLs" section (where clients verify features)
    - **Expected outcomes**: What the reviewer should see when following the steps
    - Use client-friendly language (no jargon)

#### If status is "Awaiting Approval" or "Need more information":

The task is blocked pending client input. Generate clarification questions to gather the information needed before implementation can proceed.

1. **Understand the request**: Read the task description carefully to identify:
   - What the client is asking for
   - Any stated constraints or preferences
   - Implicit assumptions that need validation

2. **Review existing comments**: Check for:
   - Prior clarifications or partial answers
   - Questions already asked (don't repeat them)
   - Context that narrows down the scope

3. **Explore the codebase**: Use the Task tool with `subagent_type=Explore` to understand:
   - Current implementation of related features
   - Technical constraints or considerations
   - Patterns that might influence the approach
   - What's feasible vs. what requires significant changes

4. **Identify clarification needs**: Look for gaps in these categories:

   **Requirements & Acceptance Criteria**
   - What specific behavior is expected?
   - What are the success criteria?
   - Are there edge cases to consider?

   **User Experience**
   - Who will use this feature?
   - What workflow should it follow?
   - Are there visual/UI preferences?

   **Scope & Boundaries**
   - What's explicitly in scope?
   - What should be excluded?
   - Are there related changes that should wait for a future task?

   **Technical Decisions** (frame in user-friendly terms)
   - Are there multiple valid approaches? Present as options with trade-offs
   - Are there dependencies on other features or data?
   - Are there constraints the client should know about?

   **Data & Content**
   - What data should be displayed/captured?
   - Are there specific formats or validation rules?
   - Who can see/edit this data?

5. **Draft clarification questions**: Create 5-8 targeted questions that:
   - Are specific and actionable (not vague)
   - Use plain language (no technical jargon)
   - Present options where applicable (helps client decide faster)
   - Are numbered for easy reference in responses
   - Focus on decisions that block implementation

6. **Preview for user approval**: Show the questions in a formatted preview:

   > ---
   >
   > **Before we begin implementation, I have a few questions to make sure we build exactly what you need:**
   >
   > 1. **[Question about requirement]**
   >    - Option A: [Description]
   >    - Option B: [Description]
   >
   > 2. **[Question about scope]**
   >
   > 3. **[Question about user experience]**
   >
   > ...
   >
   > ---
   >
   > Should I post these questions to the Zoho task?

7. **Post to Zoho** (only after user approval):
   ```
   mcp__zoho-projects__add_task_comment(
       project_id: "<project_id>",
       task_id: "<task_id>",
       content: "<b>Before we begin implementation, I have a few questions to make sure we build exactly what you need:</b><br><br><b>1. [Question]</b><ul><li>Option A: [Description]</li><li>Option B: [Description]</li></ul><b>2. [Question]</b><br><br>..."
   )
   ```

8. **Keep status unchanged**: Do NOT change the status. The task remains in its current state ("Awaiting Approval" or "Need more information") until the client responds with answers.

**Question Quality Guidelines:**

- **Be specific**: "Should the export include all columns or just visible ones?" not "What should the export contain?"
- **Offer options**: When there are clear choices, list them with brief descriptions of trade-offs
- **Avoid yes/no when possible**: Open questions get more useful answers
- **Group related questions**: If multiple questions relate to one topic, present them together
- **Prioritize blockers**: Ask about things that would change the implementation approach first
- **Skip obvious answers**: Don't ask about things clearly stated in the description

**Example Questions by Category:**

| Category | Example Question |
|----------|------------------|
| Scope | "Should this feature apply to all users, or just managers?" |
| UX | "Would you prefer this as a button on the page, or in the dropdown menu?" |
| Data | "What information should appear in the notification - just the title, or the full details?" |
| Options | "We could show this as: (A) a popup dialog, or (B) a slide-in panel. Which feels right?" |
| Edge cases | "If a user hasn't completed their profile, should they still see this option?" |
| Priority | "Should this replace the current behavior, or work alongside it?" |

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
   - **No superfluous language**: Do NOT use filler phrases like "Implementation verified complete:", "This feature is now complete", "Here's what you can do:", "Feature verification:", "Investigation Complete:", etc. Start directly with the content.
   - **No redundant titles**: Do NOT start with "[SCREENSHOTS]", "Status update:", "Investigation Complete", "Summary:", or similar meta-titles.
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
   <ul><li>Key capability 1</li><li>Key capability 2</li></ul><b>1. [Screenshot description]:</b><br/><img src="https://previewengine-accl.zoho.com/image/WD/{file_id}?x-cli-msg={encoded_data}" /><br/><br/><b>How to verify:</b><ol><li>Visit <a href="{PREVIEW_URL}/staff/page">the page</a></li><li>Perform [specific action]</li><li>Observe [expected result]</li></ol><!-- screenshots-evidence -->
   ```

   **Note:** Replace `{PREVIEW_URL}` with the preview URL from the project's CLAUDE.md.

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
- No meta-titles ("[SCREENSHOTS]", "Feature verification:", "Status update:", "Investigation Complete:")
- No redundant headers ("Screenshots:" before images - they're obviously screenshots)
- No filler phrases ("This feature is now complete and ready for your review")
- No status-announcing openings ("Investigation Complete", "Update:", "Summary:")
- Jump straight to what matters: findings, evidence, recommendations, or verification steps

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

### Scoping Questions (For "Awaiting Approval")

```html
<b>Before we begin implementation, I have a few questions to make sure we build exactly what you need:</b><br><br><b>1. [Requirement question]</b><br>[Context or options if applicable]<br><br><b>2. [Scope question]</b><ul><li>Option A: [Description]</li><li>Option B: [Description]</li></ul><b>3. [UX question]</b><br><br><b>4. [Data/content question]</b><br><br><b>5. [Priority/edge case question]</b><br><br>Once I have your answers, I can proceed with implementation.<!-- scoping-questions -->
```

**Note:**
- Include the `<!-- scoping-questions -->` marker at the end for future reference
- Number questions for easy reference in client responses
- Use `<ul><li>` for options, plain `<br>` for simple questions
- Keep questions concise but specific

### Complete Example (For "To do" → "Open")

```html
Code ready for internal review:<ul><li>Branch: <code>feature/XXX-TXXX-csv-export</code></li><li>Pull Request: <a href="{BITBUCKET_PR_URL}">PR #45</a></li></ul><b>What this adds:</b><ul><li>Staff can now download session data as a spreadsheet file</li><li>The download includes all visible columns and respects any active filters</li></ul><b>How to verify (once deployed):</b><ol><li>Go to <a href="{PREVIEW_URL}/staff/sessions">the Sessions page</a></li><li>Click the "Export" button in the top right</li><li>A spreadsheet file should download to your computer</li><li>Open the file to confirm it contains the session data</li></ol>
```

### Complete Example (For "Open"/"In Review" with screenshots)

```html
<ul><li>Staff can now download session data as a spreadsheet file</li><li>The download includes all visible columns and respects any active filters</li></ul><b>Where to find it:</b><br/>Visit <a href="{PREVIEW_URL}/staff/sessions">the Sessions page</a> and look for the "Export" button in the top right.<br/><br/><img src="https://previewengine-accl.zoho.com/image/WD/abc123?x-cli-msg=xyz" /><br/><br/><b>How to verify:</b><ol><li>Visit <a href="{PREVIEW_URL}/staff/sessions">the Sessions page</a></li><li>Click the "Export" button</li><li>A spreadsheet file should download to your computer</li><li>Open the file to confirm it contains the session data</li></ol><!-- screenshots-evidence -->
```

**Note:** Replace placeholders with actual values:
- `{PREVIEW_URL}` - The preview URL from project's CLAUDE.md "Environment URLs" section (where clients verify features)
- `{BITBUCKET_PR_URL}` - The actual PR URL from BitBucket

## Final Checklist

### For "Awaiting Approval" or "Need more information" Tasks (Scoping/Clarification)

Before posting clarification questions:

- [ ] **Read task description**: Understand what the client is asking for
- [ ] **Check existing comments**: Look for prior context or partial answers
- [ ] **Explore codebase**: Understand technical considerations that inform questions
- [ ] **Questions are specific**: Each question is actionable, not vague
- [ ] **No jargon**: Questions use plain language a non-technical client can understand
- [ ] **Options provided**: Where applicable, present choices with brief descriptions
- [ ] **5-8 questions max**: Focused on decisions that block implementation
- [ ] **No duplicates**: Questions not already asked in previous comments
- [ ] **Preview approved**: User has seen and approved the questions
- [ ] **Hidden marker included**: Comment ends with `<!-- scoping-questions -->` for tracking
- [ ] **Status unchanged**: Keep current status (do NOT change it)

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
  - [ ] **Verification steps**: Numbered steps to test (using project's preview URL from CLAUDE.md)
  - [ ] **CLICKABLE LINK**: At least one `<a href="...">` link to the specific page where the feature can be tested (MANDATORY)
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
- [ ] **CLICKABLE LINK (MANDATORY)**: At least one `<a href="{PREVIEW_URL}/specific/path">` in verification steps (use URL from project's CLAUDE.md)
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

- **CLICKABLE LINKS ARE MANDATORY**: Every comment with verification steps MUST include at least one `<a href="...">` link to the specific page where the feature can be tested. Never describe navigation without providing a direct link.
- **Project ID**: Check project's CLAUDE.md "Zoho Projects Configuration" section
- **Open Status ID**: Get from project's CLAUDE.md "Zoho Projects Configuration" section (varies by project)
- **Never set "In Review"**: The skill should never change a task to "In Review" status - that is reserved for when the client can see the feature on production/testing and is done manually
- **Awaiting Approval**: Use when task is blocked pending client clarification
- **Preview URL**: Each project defines its preview URL in CLAUDE.md under "Environment URLs" - ALWAYS use this URL for client verification links in Zoho comments
- **BitBucket Repo**: Check project's CLAUDE.md or git remote for workspace/repo-slug
- **Screenshots are gitignored**: Don't commit screenshot images to the repo (only the test files)
- **Single-line comments**: Zoho converts newlines to `<br>` tags - keep entire comment on one line
- **No emojis**: Don't use checkmarks (✅) or other emojis in Zoho comments
- **No dates**: Don't include "Status Update - January 2026" headers
- **Screenshot hosting**: Upload to Zoho first to get URLs, then embed in BitBucket PR description
