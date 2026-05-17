# Data Schema Reference

## AEM Operational Telemetry (OpTel) Schema

OpTel is Adobe Experience Manager's built-in real user monitoring system. It collects sampled
pageview data using checkpoints — named events during page load and interaction.

### Core Fields

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique pageview identifier |
| timestamp | ISO 8601 | When the event occurred |
| date | YYYY-MM-DD | Date only |
| hour | integer | Hour of day (0–23) |
| domain | string | Site domain |
| url | string | Page path |
| device_type | enum | desktop, mobile, bot |
| operating_system | string | Windows, macOS, Linux, iOS, Android |
| referrer | string | Traffic source domain or "direct" |
| language | string | Browser language (e.g., en-US) |
| sampling_weight | integer | Each row represents N real pageviews (typically 100) |

### Core Web Vitals

| Field | Type | Thresholds |
|-------|------|------------|
| lcp_ms | float | Good: ≤2500 · Needs improvement: ≤4000 · Poor: >4000 |
| cls | float | Good: ≤0.1 · Needs improvement: ≤0.25 · Poor: >0.25 |
| inp_ms | float | Good: ≤200 · Needs improvement: ≤500 · Poor: >500 |
| ttfb_ms | float | Time to First Byte in milliseconds |
| lcp_rating | enum | good, needs-improvement, poor |
| cls_rating | enum | good, needs-improvement, poor |
| inp_rating | enum | good, needs-improvement, poor |

Note: iOS devices do not report Core Web Vitals due to browser limitations.

### Engagement

| Field | Type | Description |
|-------|------|-------------|
| bounce | boolean | Visitor left without clicking |
| engaged | boolean | Visitor consumed content or clicked |
| time_on_page_s | float | Seconds on page |
| is_organic | boolean | Traffic from organic sources |

### Interaction

| Field | Type | Description |
|-------|------|-------------|
| clicked | boolean | A click event occurred |
| click_source | string | CSS selector of clicked element |
| click_target | string | URL or element that was clicked |
| blocks_viewed | string | Pipe-delimited list of block class names |
| lcp_element | string | CSS selector of LCP element |
| navigate_from | string | Internal page that referred this view |

### Errors and Forms

| Field | Type | Description |
|-------|------|-------------|
| has_error | boolean | JS error occurred |
| error_message | string | Error text |
| form_fill | boolean | A form field was filled |
| form_submit | boolean | A form was submitted |
| is_404 | boolean | Page returned 404 |
| four04_url | string | URL that was requested (if 404) |

### OpTel Checkpoints Quick Reference

| Checkpoint | When it fires |
|------------|---------------|
| top | Page loading begins |
| cwv | Core Web Vitals reading collected |
| lcp | Largest Contentful Paint rendered |
| viewblock | Block scrolled into viewport |
| viewmedia | Image/video scrolled into viewport |
| navigate | Internal navigation |
| enter | External referrer entry |
| click | Any click event |
| error | Unhandled JS error |
| 404 | Page not found |
| fill | Form field filled |
| formsubmit | Form submitted |
| search | Site search performed |
| consent | Consent banner interaction |
| language | Content language detected |
| a11y | Accessibility feature detected |
| acquisition | Inorganic traffic source |
| redirect | Redirect hops counted |
| loadresource | Fragment/JSON API loaded |

---

## Medallia Voice-of-Customer Schema

| Field | Type | Description |
|-------|------|-------------|
| response_id | string | Unique response identifier (e.g., MED-5001) |
| timestamp | ISO 8601 | When the response was submitted |
| date | YYYY-MM-DD | Date only |
| channel | enum | web_intercept, email_survey, post_purchase, app_feedback |
| page_url | string | Page where feedback was triggered |
| device | enum | mobile, desktop |
| csat_score | integer | Customer Satisfaction (1–5, higher = better) |
| nps_score | integer | Net Promoter Score (0–10, higher = better) |
| ces_score | integer | Customer Effort Score (1–7, higher = MORE effort = worse) |
| verbatim | string | Free-text customer feedback |
| journey_stage | enum | checkout, build, signup, login, browse |
| sentiment | enum | positive, neutral, negative |
| resolved | boolean | Whether the issue was resolved |

### Interpreting Scores

| Metric | Good | Neutral | Bad |
|--------|------|---------|-----|
| CSAT | 4–5 | 3 | 1–2 |
| NPS | 9–10 (Promoter) | 7–8 (Passive) | 0–6 (Detractor) |
| CES | 1–2 (Low effort) | 3–4 | 5–7 (High effort) |

---

## AirTable Target Schema (Requirements Output)

| Column | Type | Description |
|--------|------|-------------|
| Summary | Single line text | Ticket title — specific and actionable |
| Assignee | Single line text | Dev assigned (leave blank if unknown) |
| Reporter | Single line text | Always "SLICC Agent" |
| Work Type | Single select | Bug, Story, or Task |
| Priority | Single select | Critical, High, Medium, Low |
| Description | Long text | Structured markdown: Problem, Impact, Repro Steps, Technical Details, Acceptance Criteria, Loom link |
| Attachments | URL | Loom share URL |
| Attachment Summary | Single line text | One-sentence description of the recording |
