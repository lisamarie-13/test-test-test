# Glitchcraft Margarita Labs — UX Research Audit

An autonomous UX research workflow powered by SLICC. Analyzes operational telemetry
and voice-of-customer data, records broken user journeys via Loom, writes requirements,
and posts everything to AirTable.

## Folder Structure

```
glitchcraft-ux-audit/
├── README.md                          ← You are here
├── ux-research-agent/                 ← SLICC Skill
│   ├── SKILL.md                       ← Main skill file (the playbook)
│   └── references/
│       └── data-schemas.md            ← OpTel + Medallia + AirTable schema reference
└── data/                              ← Synthetic demo data
    ├── glitchcraft_optel_events.csv           ← 31,380 OpTel checkpoint events (30 days)
    ├── glitchcraft_medallia_verbatims.csv     ← 533 Medallia VoC responses
    └── glitchcraft_margarita_labs_optel_data.xlsx  ← Combined workbook (4 sheets)
```

## Setup Instructions

### Step 1: Get SLICC running

The macOS .dmg is the most reliable path right now:

1. Download from: https://www.sliccy.ai/download/slicc.dmg
2. Open the .dmg, drag SLICC to Applications
3. Double-click to launch — it opens its own sandboxed Chrome profile (no access to your personal sessions)

If you prefer the CLI, make sure `npx sliccy` actually launches a workspace (you should see
the SLICC UI, not just a blank Chrome tab). If it silently returns to the prompt, the .dmg
is the better option.

### Step 2: Install the skill into SLICC

**Option A — Drag and drop (easiest)**

1. In the SLICC workspace, find the file drop area (paperclip button or drag onto the chat)
2. Drag the entire `ux-research-agent` folder into the workspace
3. SLICC should detect the SKILL.md and install it to `/workspace/skills`

**Option B — Via the SLICC shell**

If SLICC is running and you have shell access, you can ask SLICC to install the skill:

```
upskill /path/to/glitchcraft-ux-audit/ux-research-agent
```

Or ask SLICC in chat:
> "Install the skill from the ux-research-agent folder I just dropped in"

**Option C — Manual placement**

If SLICC discovers skills from your filesystem, copy the skill folder:

```bash
# If SLICC reads from ~/.claude/skills (compatible mode)
cp -r ux-research-agent ~/.claude/skills/

# Or if SLICC reads from ~/.agents/skills
cp -r ux-research-agent ~/.agents/skills/
```

### Step 3: Load the data into AirTable

1. Go to AirTable and create a new base called "Glitchcraft UX Audit"
2. Import `data/glitchcraft_optel_events.csv` as a new table named "OpTel Events"
3. Import `data/glitchcraft_medallia_verbatims.csv` as a new table named "Medallia Verbatims"
4. Create a third table called "Requirements" with these columns:

| Column Name | Field Type |
|-------------|-----------|
| Summary | Single line text |
| Assignee | Single line text |
| Reporter | Single line text |
| Work Type | Single select (Bug, Story, Task) |
| Priority | Single select (Critical, High, Medium, Low) |
| Description | Long text (enable rich text) |
| Attachments | URL |
| Attachment Summary | Single line text |

### Step 4: Set up browser sessions

In the SLICC browser (the sandboxed Chrome), log into:

- [ ] **AirTable** — Open the "Glitchcraft UX Audit" base
- [ ] **Loom** — Install the Loom Chrome extension and log in
- [ ] **Glitchcraft Margarita Labs site** — (once built/deployed, open it in a tab)

### Step 5: Run the skill

In the SLICC chat, type:

> "Analyze the OpTel and Medallia data in AirTable for glitchcraftmargaritalabs.com,
> find the broken user journeys, record them with Loom, write requirements, and post
> them to the Requirements table."

Or use the slash command if available:

```
/ux-research-agent
```

The agent will:
1. Open the AirTable data tabs and gather analytics
2. Present a ranked list of broken journeys and ask for your confirmation
3. Record each journey on the live site using Loom
4. Write structured requirements
5. Create AirTable records with Loom links attached

## The Four Baked-In Broken Journeys

The synthetic data has four intentionally broken pages for SLICC to discover:

| Page | Issue | Bounce | Errors | LCP |
|------|-------|--------|--------|-----|
| `/order/checkout` | Address form clears on validation error | 72% | 15% | 5,721ms |
| `/rewards/signup` | Email validation rejects valid emails | 64% | 11% | 3,470ms |
| `/rewards/login` | Login silently fails, no error feedback | 58% | 10% | 3,126ms |
| `/order/build-your-own` | Price doesn't update; layout shifts | 56% | 7% | 4,496ms |

## What's Still Needed

- [ ] **Glitchcraft Margarita Labs website** — A live site with these broken journeys baked in for SLICC to walk through and record
- [ ] **SLICC running successfully** — The npm install issue needs to be resolved (try the .dmg)
- [ ] **End-to-end test** — Run the full workflow once to tune timing, Loom interaction, and AirTable field mapping
