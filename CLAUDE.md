## Skills
在处理任何前端、UI、页面相关任务时，请先阅读 .claude/skills/skills/frontend-design/SKILL.md。

---

# Ongredients × Influu — Affiliate Dashboard

## Project Overview

A static, client-side affiliate marketing dashboard for **Ongredients** (Korean skincare brand) built as a GitHub Pages site. The dashboard tracks TikTok Shop affiliate performance — commission-based creator outreach, paid affiliate management, and content strategy — and is presented weekly to the client.

- **Main file:** `ongredients.html` (single-page app, all CSS/JS inline)
- **Style file:** `style.css` (shared with `index.html`)
- **Data directory:** `data/ongredients/` (all JSON)
- **Language:** Dashboard content in English (client is Korean); communicate with user in Chinese

---

## Architecture

### Tab System
The page has three top-level tabs:
1. **Commission Based Creators** (`panel-commission`) — core funnel tracking
2. **Paid Affiliates** (`panel-paid`) — curated paid creator management
3. **Content & Strategy** (`panel-content`) — message templates and A/B test one-pagers

### Data Loading
All JSON loaded at `init()` via `Promise.all()` on page load:
- `invitation.json` + `video_daily.json` loaded immediately (used in Commission tab)
- `video_detail.json`, `paid_affiliates.json`, `shipped_creators.json`, `posted_videos.json` fetched on demand or in init

---

## Data Files

All files live in `data/ongredients/`.

### `invitation.json`
Daily funnel metrics from EUKA AI (Influu agents only, Pacific Time).

| Field | Description |
|-------|-------------|
| `date` | YYYY-MM-DD (Pacific Time) |
| `sent` | Invitations sent that day |
| `req` | Sample requests received |
| `sample_approved` | Samples shipped (renamed field; this = shipped) |
| `ready_to_ship` | Always 0 (deprecated, kept for schema compat) |
| `approved` | Invitations accepted (collab confirmed) |
| `posted` | Videos posted (`new_videos_detected` from EUKA) |

> **Important:** `sample_approved` in this file = "Sample Shipped" in the UI. The field name is legacy; the dashboard reads it as shipped.

### `video_daily.json`
Daily video performance aggregates (Aug 2025 – present).

| Field | Description |
|-------|-------------|
| `date` | YYYY-MM-DD |
| `label` | Human-readable date label |
| `videos` | New videos detected that day |
| `impressions` | Shoppable video impressions |
| `gmv` | Affiliate GMV ($) |
| `orders` | Affiliate orders |
| `avg_ctr` | Average video CTR (decimal, e.g. 0.082 = 8.2%) |
| `avg_ctor` | Average video CTOR (decimal) |

### `video_detail.json`
Individual video performance records. Keys match original EUKA CSV export format:

| Key | Description |
|-----|-------------|
| `Video name` | Title of TikTok video |
| `Video link` | TikTok URL |
| `Creator username` | TikTok handle (no @) |
| `Video post date` | Date string |
| `Shoppable video impressions` | Impressions count |
| `GMV` | Revenue ($) |
| `Affiliate orders` | Order count |
| `Affiliate CTR` | CTR value (may be null) |
| `Shoppable video GPM` | GPM value |

### `shipped_creators.json`
Creators who received samples through Influu campaigns.

| Field | Description |
|-------|-------------|
| `handle` | TikTok handle (no @) |
| `url` | TikTok profile URL |
| `shipped_date` | Pacific Time date |
| `req_date` | Sample request date |
| `campaign` | EUKA campaign name (e.g. "Influu Agent2 EX Lotion BPC") |
| `gmv_30d` | Creator's 30-day GMV from EUKA profile |
| `level` | Calculated level (L1–L7) |

### `posted_videos.json`
43 videos posted through Influu campaigns (new_videos_detected only).

| Field | Description |
|-------|-------------|
| `video_id` | EUKA video ID |
| `thumbnail` | URL: `https://database.euka.ai/storage/v1/object/public/creator_videos_photos/{video_id}.webp` |
| `title` | Video caption/title |
| `url` | TikTok video URL |
| `creator` | TikTok handle |
| `creator_url` | TikTok profile URL |
| `post_date` | Pacific Time date (YYYY-MM-DD) |
| `gmv` | Revenue ($) |
| `views` | View count |
| `likes` | Like count |
| `product` | Product name |
| `agent` | Influu agent name |

### `paid_affiliates.json`
Static data for paid affiliate creator list. Fields include:
`name`, `group` (Confirmed/Backup/Rejected), `level`, `gmv`, `gmv_display`, `followers`, `views`, `category`, `url`, `gender`, `age`, `ethnicity`, `traits`, `selling_categories`, `audience_range`, `female_pct`, `male_pct`, `notes`

> **Section 4 (Paid Affiliates) is static — do NOT replace with API data.**

---

## EUKA MCP Integration

### Store
- **Store:** Ongredients US
- **Store ID:** `47c60ce7-23a8-43ef-9ecd-739cbcf2ae76`
- **MCP Tool:** `mcp__claude_ai_EUKA_AI__query_store_data`

### Influu Agents (Commission-Based Only)
Invitation data uses only these 6 agents (all start with "Influu"):
- `Influu Agent1 EX Lotion BPC`
- `Influu Agent2 EX Lotion BPC`
- `Influu Agent3 EX Lotion BPC — Premium`
- (+ 3 others with "Influu" prefix)

Filter: `agent_name ILIKE 'Influu%'`

### Timezone
**EUKA returns UTC. All data must be converted to Pacific Time (UTC-7) before grouping by date.**
- Use `AT TIME ZONE 'America/Los_Angeles'` in EUKA SQL queries
- Verified: `sample_requests` on 2026-05-01 = 59 in Pacific Time (was 53 in UTC — wrong)

### Key Metrics Mapping
| Dashboard Label | EUKA Field | Notes |
|-----------------|-----------|-------|
| Invitations Sent | `invites_sent` | daily |
| Sample Requested | `sample_requests` | daily |
| Sample Shipped | `shipped` | field definition unclear, pending EUKA support |
| Invitations Accepted | `invites_accepted` | daily |
| Video Posted | `new_videos_detected` | NOT `completed_status_changes` (that's 118, too many) |

### Video Count Clarification
- `new_videos_detected` = **43** ✓ matches EUKA website UI (correct metric)
- `completed_status_changes` = 118 (TikTok batch status updates, not actual new posts — wrong)

### TikTok Thumbnail URLs
`https://database.euka.ai/storage/v1/object/public/creator_videos_photos/{video_id}.webp`

### Creator Level Tiers (GMV 30d)
| Level | GMV Range |
|-------|-----------|
| L1 | < $5K |
| L2 | $5K – $25K |
| L3 | $25K – $60K |
| L4 | $60K – $150K |
| L5 | $150K – $400K |
| L6 | $400K – $1.5M |
| L7 | > $1.5M |

---

## Commission Tab — Section Details

### Section 01: Invitation & Sample Funnel

**5 stat cards:**
1. **Invitations Sent** (blue #3b82f6)
2. **Sample Requested** (light blue #60a5fa)
3. **Sample Shipped** (green #22c55e) — **clickable → Shipped Creators modal**
4. **Approval Rate** (grey #94a3b8) — `Sample Shipped ÷ Sample Requested × 100%`, hover shows tooltip explaining methodology
5. **Video Posted** (yellow #f5c842) — **clickable → Posted Videos modal**

**Chart:** Line chart, 4 toggleable metrics (Invitations Sent, Sample Requested, Sample Shipped, Video Posted)

**Date range:** Derived dynamically from `invitation.json` first/last entry dates.

**Cross-modal date sharing:**
```js
window._s1Start = currentStart;
window._s1End = currentEnd;
```
Both modals read these globals to filter by the currently displayed date range.

### Section 02: Video Activity & Channel Performance

**4 KPI cards:** Total Videos, New Videos (in range), Video Impressions (in range), Affiliate Orders (+ avg CTR)

**Charts:**
- `chartVideoAct`: New Videos vs Cumulative Total Videos (dual-axis)
- `chartCVRChan`: Video CTR, Video CTOR, Impressions (dual-axis)

### Top Performing Videos

Table leaderboard, below Section 02. Rank-by filter: GMV / CTR / GPM. Shows top 20.
**Columns:** # | Video | Creator | Post Date | GMV | CTR | GPM | Orders | Impressions

### Section 03: Affiliate GMV
Single GMV line chart with date range. No goal set currently.

---

## Paid Affiliates Tab — Section Details

### Section PA-01: Creator Table
- **Groups:** Confirmed Deals → Backup Options → Rejected
- **Filters:** Group, Level (L3–L6), Category
- **Sortable columns:** Creator, Level, GMV, Followers
- **Per-row actions:** View Details, Approve, Reject
- **Persistence:** Approvals, rejections, and comments saved to `localStorage`
  - Keys: `ongredients_pa_approved`, `ongredients_pa_comments`, `ongredients_pa_rejected`

### Section PA-02: Video Performance (Paid)
Chart tracking CTR/CTOR over 12-week window.

### Section PA-03: Weekly Sales & Budget
Budget % attainment and GMV % attainment cards with progress bars.

---

## Modals

### Shipped Creators Modal (`shipped-modal-overlay`)
- **Trigger:** Sample Shipped stat card
- **Features:** Search by handle, filter by level (only levels present in data), sorted by GMV desc
- **Row shows:** @handle (link), GMV 30d, Level badge, Shipped date, Agent name

### Posted Videos Modal (`posted-modal-overlay`)
- **Trigger:** Video Posted stat card
- **CSS class:** `ong-modal-wide` (max-width 900px, overrides default 560px)
- **Table columns:** Thumbnail | Creator | Title | Date | Revenue | Views | Likes
- **Sortable:** Date (default desc), Revenue, Views, Likes — click header to sort, click again to toggle asc/desc
- **Sort state:** `_postedSort = { col, dir }`, resets to `{ col: 'date', dir: 'desc' }` on each open

### Creator Detail Modal (`pa-modal-overlay`)
- **Trigger:** "View Details" in Paid Affiliates table
- **Sections:** Key metrics, Why We Recommend, Creator Profile, Audience split, Notes textarea, Approve/Reject buttons

---

## CSS Design System

```css
--ong-yellow:      #f5c842
--ong-yellow-deep: #c89800
--ong-yellow-pale: #fffce0
--ong-yellow-mid:  #fff6b0
--ong-dark:        #1c1a0e
--ong-text2:       #5c5640
--ong-muted:       #9a9070
--ong-border:      rgba(200,160,0,0.18)
--ong-surface:     #ffffff
```

Background: `#fefcf0` with subtle noise texture (SVG inline pattern).

**Important:** There are two `.ong-modal` CSS blocks in the file (around line 461 and line 1673). The second one (`max-width: 560px`) overrides the first. `.ong-modal-wide` must be targeted as `.ong-modal.ong-modal-wide` (double class) placed after line 1673 to override correctly.

### Hero Section
**Do not modify** the hero bottle display height, background-position, or image styling. It has been perfected.

---

## Weekly Data Update

To update all live data sections (Section 01, 02, 03), send this prompt before the weekly Tuesday meeting:

> "请帮我更新Ongredients dashboard的数据。使用EUKA MCP API（storeId: `47c60ce7-23a8-43ef-9ecd-739cbcf2ae76`）更新以下文件：
> 1. `data/ongredients/invitation.json` — 所有Influu agent的每日漏斗数据（Pacific Time）
> 2. `data/ongredients/video_daily.json` — 每日视频表现数据
> 3. `data/ongredients/video_detail.json` — 所有视频的详细表现数据
> 4. `data/ongredients/shipped_creators.json` — 已发样品的达人列表
> 5. `data/ongredients/posted_videos.json` — 已发布的视频列表（只用new_videos_detected）
> 更新完成后push到GitHub。"

**Do NOT update** `paid_affiliates.json` via API — it is managed manually.

---

## Known Issues / Notes

- **Sample Shipped数据**: EUKA的`shipped`字段定义待确认，数值与EUKA网站不一致，已联系EUKA support。
- **Approval Rate tooltip**: 显示文字为 "Approval Rate = Sample Shipped ÷ Sample Requested. Note: Accuracy improves once all pending samples are shipped."
- **Level filter buttons**: 动态生成，只显示数据中实际存在的level，不显示空level。

---

## Git & Deployment

- **Repo:** `darrending0919.github.io` (GitHub Pages)
- **Branch:** `main`
- **Live URL:** `https://darrending0919.github.io/ongredients.html`
- Push to `main` auto-deploys via GitHub Pages.
