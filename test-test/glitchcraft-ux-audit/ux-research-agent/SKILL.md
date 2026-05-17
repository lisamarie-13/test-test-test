---
name: ux-research-agent
description: Autonomous UX research agent that analyzes web analytics and voice-of-customer data to discover broken user journeys, screen-records the worst ones via Loom, writes structured requirements to fix them, and posts everything to AirTable. Use when the user asks to find UX problems, analyze user journeys, audit a website for usability issues, create bug reports from analytics data, or record broken user flows. Requires browser access to AirTable, Loom Chrome extension, and the target website.
---

# UX Research Agent

An autonomous workflow that turns raw operational telemetry and customer feedback into prioritized, recorded, and documented bug tickets — ready for a dev team to pick up.

## Overview

This skill performs five phases:

1. **Gather** — Pull analytics (OpTel) and voice-of-customer (Medallia) data from AirTable or another browser-accessible source
2. **Analyze** — Cluster pain points, identify broken user journeys, and rank them by severity
3. **Record** — Walk through each broken journey on the live site while Loom records the screen
4. **Write** — Generate structured requirements (summary, description, priority, work type) for each journey
5. **Deliver** — Create one AirTable record per broken journey with the Loom link and requirements attached

## Prerequisites

Before running this skill, confirm the following are available in the current browser session:

- [ ] **AirTable** — Logged in, with the source data base open in a tab AND the target requirements base accessible
- [ ] **Loom Chrome Extension** — Installed and logged in (the extension icon should be visible in the toolbar)
- [ ] **Target website** — The site to be audited is publicly accessible at a known URL
- [ ] **Data is loaded** — The OpTel events table and Medallia verbatims table exist in AirTable (or the designated data source)

If any prerequisite is missing, stop and tell the user what to set up before proceeding.

## Phase 1: Gather Data

### From AirTable (default source)

1. Open the AirTable base containing the analytics data
2. Navigate to the **OpTel Events** table (or whatever the user specifies)
3. Apply a view or filter that shows the last 30 days of data
4. Note the key columns. The expected schema is:

| Column | Description |
|--------|-------------|
| `url` | Page path (e.g. `/order/checkout`) |
| `device_type` | desktop, mobile, or bot |
| `lcp_ms` | Largest Contentful Paint in milliseconds |
| `lcp_rating` | good, needs-improvement, or poor |
| `cls` | Cumulative Layout Shift score |
| `cls_rating` | good, needs-improvement, or poor |
| `inp_ms` | Interaction to Next Paint in milliseconds |
| `inp_rating` | good, needs-improvement, or poor |
| `bounce` | TRUE if the visitor bounced |
| `has_error` | TRUE if a JS error occurred |
| `error_message` | The error string, if any |
| `form_fill` | TRUE if a form field was filled |
| `form_submit` | TRUE if a form was submitted |
| `time_on_page_s` | Seconds spent on page |

5. Navigate to the **Medallia Verbatims** table
6. Note these columns:

| Column | Description |
|--------|-------------|
| `page_url` | The page the feedback was triggered on |
| `csat_score` | Customer satisfaction (1–5) |
| `nps_score` | Net Promoter Score (0–10) |
| `ces_score` | Customer Effort Score (1–7, higher = harder) |
| `verbatim` | Free-text customer feedback |
| `sentiment` | positive, neutral, or negative |
| `journey_stage` | checkout, build, signup, login, browse, etc. |

7. Extract or screenshot enough data to perform the analysis. For large tables, focus on:
   - Pages with the highest bounce rates
   - Pages with the highest error rates
   - Pages with poor CWV ratings (LCP > 4000ms, CLS > 0.25, INP > 500ms)
   - Medallia verbatims with negative sentiment and CSAT ≤ 2

### From other sources

If the data lives in Snowflake, Google Sheets, or another browser-accessible tool, open it in a tab, apply equivalent filters, and extract the same fields. The analysis phase uses the same schema regardless of source.

## Phase 2: Analyze and Rank Journeys

Using the gathered data, identify broken user journeys by clustering problems around pages and flows.

### Scoring criteria

For each candidate page or flow, calculate a **severity score** (higher = worse):

```
severity = (bounce_rate × 30) + (error_rate × 25) + (poor_lcp_pct × 15) + (poor_cls_pct × 10) + (poor_inp_pct × 10) + (neg_medallia_pct × 10)
```

Where:
- `bounce_rate` = fraction of pageviews that bounced (0–1)
- `error_rate` = fraction of pageviews with JS errors (0–1)
- `poor_lcp_pct` = fraction of pageviews with LCP rating "poor" (0–1)
- `poor_cls_pct` = fraction with CLS rating "poor" (0–1)
- `poor_inp_pct` = fraction with INP rating "poor" (0–1)
- `neg_medallia_pct` = fraction of Medallia responses for this page with sentiment "negative" (0–1)

### Prioritization

Rank all candidate pages by severity score descending. Assign priority:

| Severity Score | Priority |
|----------------|----------|
| ≥ 25 | Critical |
| 15–24 | High |
| 8–14 | Medium |
| < 8 | Low |

### Journey grouping

Some broken pages are part of the same journey. Group them:
- `/order` → `/order/build-your-own` → `/order/checkout` → `/order/confirmation` is the **Order Journey**
- `/rewards` → `/rewards/signup` and `/rewards/login` is the **Rewards Journey**
- Standalone pages are their own journey

Select the **top 3–5 highest-priority journeys** to record and document. If the user specifies a number, use that instead.

### Output of this phase

Before proceeding, present the ranked findings to the user in a summary like:

```
BROKEN JOURNEY ANALYSIS
=======================
1. Order Checkout (/order/checkout) — CRITICAL
   Bounce: 72% | Errors: 15% | LCP: 5721ms (poor) | CLS: 0.45 (poor)
   Top Medallia theme: "Address form clears on validation error"
   CSAT: 2.0/5 | 148 negative verbatims

2. Rewards Signup (/rewards/signup) — CRITICAL
   Bounce: 64% | Errors: 11% | LCP: 3470ms (poor)
   Top Medallia theme: "Email validation rejects valid addresses"
   CSAT: 1.9/5 | 77 negative verbatims

3. Build Your Own (/order/build-your-own) — HIGH
   ...

Proceed with recording these journeys? [Y/N]
```

Wait for user confirmation before proceeding to Phase 3.

## Phase 3: Record Broken Journeys with Loom

For each broken journey (in priority order):

### Step 1: Prepare the recording

1. Open the target website in a new tab
2. Navigate to the starting page of the journey (e.g. `/order` for the checkout journey)
3. Click the Loom Chrome Extension icon in the toolbar
4. Select **Current Tab** as the recording mode
5. Select **Screen Only** (no camera) unless the user requests camera
6. Click **Start Recording**
7. Wait for the 3-second countdown to finish

### Step 2: Walk the broken journey

1. Narrate each step by pausing briefly on key elements (the recording is visual, not audio — the pauses create a natural viewing rhythm)
2. Perform the actions a real user would:
   - Fill out forms with realistic test data
   - Click buttons and navigation links
   - Scroll to show layout shifts or missing content
   - Trigger the known error conditions (e.g., enter an email with a `+` character if the bug is email validation)
3. When the error or problem occurs, pause for 3–5 seconds so viewers can see it clearly
4. If the journey spans multiple pages, navigate through each step naturally

### Step 3: Stop and save

1. Click the red stop button on the Loom recording controls
2. Wait for Loom to process and generate the shareable link
3. Copy the Loom share URL (Loom auto-copies to clipboard after recording)
4. Save the URL — you will need it in Phase 5

### Repeat

Repeat Steps 1–3 for each journey to be recorded. Each journey gets its own Loom video.

## Phase 4: Write Requirements

For each broken journey, generate a structured requirement using this exact format:

### Summary (one line)

A clear, specific title. Examples:
- "Checkout address form clears all fields on validation error"
- "Rewards signup email validation rejects valid email formats"
- "Build-your-own ingredient selector does not update price total"

### Work Type

Choose one:
- **Bug** — Something is broken and was likely working before
- **Story** — A new behavior or capability is needed
- **Task** — Technical work that doesn't change user-facing behavior

Most broken journeys will be Bugs.

### Priority

Use the priority assigned in Phase 2 (Critical, High, Medium, Low).

### Description

Write a structured description with these sections:

```markdown
## Problem Statement
[1-2 sentences describing what is broken from the user's perspective]

## Impact
- Bounce rate: [X%] (vs site average of [Y%])
- Error rate: [X%]
- CSAT score: [X/5]
- Affected pageviews (estimated): [N per month]
- Customer verbatim: "[Most representative quote from Medallia data]"

## Steps to Reproduce
1. Navigate to [URL]
2. [Step 2]
3. [Step 3]
4. Observe: [what goes wrong]
5. Expected: [what should happen]

## Technical Details
- LCP: [X]ms (poor, threshold: 2500ms)
- CLS: [X] (poor, threshold: 0.1)
- INP: [X]ms (poor, threshold: 200ms)
- JS Errors: [list any error messages from the data]
- Affected devices: [desktop/mobile/both]
- Affected OS: [if specific]

## Acceptance Criteria
- [ ] [Specific testable criterion 1]
- [ ] [Specific testable criterion 2]
- [ ] [Specific testable criterion 3]
- [ ] LCP on this page is under 2500ms
- [ ] CLS on this page is under 0.1
- [ ] No JS errors in browser console during the flow

## Loom Recording
[Link to the Loom video recorded in Phase 3]
```

### Reporter

Set to: `SLICC Agent`

### Attachments

The Loom share URL from Phase 3.

### Attachment Summary

A one-sentence description: "Screen recording showing [specific broken behavior] on [page/journey name]"

## Phase 5: Post to AirTable

### Target table schema

The target AirTable table has these columns:

| Column | Type | Description |
|--------|------|-------------|
| Summary | Single line text | The ticket title |
| Assignee | Single line text | Leave blank unless user specifies |
| Reporter | Single line text | "SLICC Agent" |
| Work Type | Single select | Bug, Story, or Task |
| Priority | Single select | Critical, High, Medium, Low |
| Description | Long text | Full structured description from Phase 4 |
| Attachments | URL or attachment | Loom share link |
| Attachment Summary | Single line text | One-sentence description of the video |

### Steps

1. Open the target AirTable base (the user will specify which base and table)
2. For each broken journey (in priority order):
   a. Click the **+** button to create a new record
   b. Fill in each column with the values from Phase 4
   c. For the Description field, paste the full markdown-formatted description
   d. For Attachments, paste the Loom URL
   e. Confirm the record is saved
3. After all records are created, take a screenshot of the completed table view showing all new records

### Alternative: Post to Trello

If the user specifies Trello instead of AirTable:

1. Open the target Trello board
2. For each broken journey, create a new card in the appropriate list (e.g., "Backlog" or "To Do")
3. Set the card title to the Summary
4. Add the Description as the card description
5. Add the Priority as a label (Critical = red, High = orange, Medium = yellow, Low = green)
6. Attach the Loom URL
7. Add a comment with the Attachment Summary

## Error Handling

- **Loom extension not responding**: Try clicking the extension icon again. If it still doesn't respond, check that the extension is enabled in `chrome://extensions`. Tell the user if it requires re-login.
- **AirTable table doesn't match schema**: If the target table has different column names, map the fields as closely as possible and tell the user about any mismatches.
- **Target website is down**: Report to the user and skip to the next journey. Do not record a broken page that is entirely unreachable.
- **Insufficient data**: If fewer than 100 sampled pageviews exist for a page, flag it as "low confidence" in the analysis and note the small sample size in the description.

## Guidelines

- Always wait for user confirmation after the analysis phase before recording
- Record journeys in priority order (Critical first)
- Keep Loom recordings under 2 minutes per journey — focus on the specific breakage, not a full site tour
- Use realistic but obviously fake test data in form fields (e.g., "Jane Doe", "123 Test Street", "test@example.com")
- Do not enter real credit card numbers or personal information
- If a journey requires authentication, ask the user to provide test credentials or pre-authenticate in the browser
- Take screenshots at key moments as backup in case Loom fails
- The Description field should be detailed enough that a developer who has never seen the site can understand and reproduce the issue
