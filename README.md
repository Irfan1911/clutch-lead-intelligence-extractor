# Clutch Lead Scraper — Automated B2B Company Data Extraction

Clutch Lead Scraper is a production Playwright-based web scraper that extracts structured company information from Clutch.co across multiple industries and niches. The tool is designed to feed into downstream email enrichment workflows, generating pre-labeled, deduplicated datasets ready for lead qualification and outreach.

The project was built to explore how intelligent data collection and workflow automation can eliminate the friction of manual B2B prospecting research.

## Problem

Manual lead research on business directories is repetitive and does not scale:

1. Visiting each company listing individually to collect basic information
2. Cross-referencing company names and websites to avoid duplicates
3. Manually organizing results across multiple spreadsheets
4. Re-running searches weeks later produces overlapping datasets with no version control
5. Integrating scraped data into downstream enrichment workflows requires manual intervention

For solopreneurs and agencies building cold email campaigns, this friction means **lost time to data collection instead of outreach**.

## What It Does

1. **Target & Scrape** — User specifies a Clutch.co niche URL and page count
2. **Extract** — Browser loads each listing page and extracts company metadata (name, description, website, location, team size, hourly rate, minimum project size)
3. **Deduplicate** — Intelligent filtering removes duplicate company names and website URLs
4. **Chunk & Export** — Results are split into 60-record CSV files with niche + timestamp naming for easy tracking
5. **Integrate** — CSV files are uploaded to Google Drive and fed into email extraction workflows

## Key Technical Highlights

- **Server-Side Pagination Handling** — Correctly identifies and iterates through true server-side pagination (not client-side rendering tricks), ensuring complete dataset coverage across large result sets
- **Intelligent Deduplication** — Dual-layer deduplication using both company name and website URL as keys prevents duplicate records from polluting downstream enrichment pipelines
- **Retry & Error Recovery** — Implements page load retry logic with extended timeouts (300s) to gracefully handle network hiccups and DOM delays
- **Chunked CSV Output** — Splits large result sets into 60-record files with timestamped naming (`niche-YYYY-MM-DDTHH-MM-N.csv`) to prevent Google Drive name collisions across multiple scrape runs
- **Stealth User-Agent Rotation** — Randomizes browser user agents to reduce detection risk on rate-limited endpoints
- **CSV Escaping** — Properly handles embedded quotes and special characters in company descriptions using standard CSV escaping (`""`), ensuring clean downstream parsing

## Tech Stack

**Automation:** Playwright (Chromium)  
**Data Processing:** Node.js  
**Output Format:** CSV  
**Integration:** Google Drive (via n8n workflows)  
**Deployment:** Local / Cloud automation (n8n)

## System Architecture

The scraper is part of a larger lead generation pipeline:
Clutch Scraper (extraction)
↓
Google Drive (versioned CSV storage)
↓
Email Enricher Workflow (contact extraction via website visits)
↓
Lead Database / Outreach Pipeline

### Pagination & Extraction Flow
For each page (1 to N):
→ Load URL with retry logic
→ Wait for .provider-list-item DOM elements
→ For each listing card:
→ Extract metadata fields
→ Decode masked website URL
→ Check deduplication sets
→ Add to results if unique
→ Write chunk to timestamped CSV
→ Move to next page

## Each file contains:
- Company Name
- Description (full service offering)
- Website URL (decoded from Clutch redirect)
- Location
- Team Size
- Hourly Rate
- Minimum Project Size

## Screenshots
Clutch Source Page
(Add Screenshot)

Extraction Process
(Add Screenshot)

CSV Output
(Add Screenshot)

## Lessons Learned & Technical Decisions

1. **Server-Side vs. Client-Side Pagination** — Initial assumption of client-side pagination proved wrong. Browser console testing revealed true server-side pagination; scraper was updated to respect API-driven page loads.

2. **Website URL Decoding** — Clutch masks company websites behind redirect parameters. Solution: Extract the `u` query parameter and `decodeURIComponent()` to recover the actual URL.

3. **Deduplication Strategy** — Single-layer deduplication (name only) caused conflicts with agencies with identical names in different regions. Solution: Dual-layer checks using both name and website URL.

4. **CSV Chunking** — Writing all 200+ records to a single file caused downstream n8n memory issues during enrichment. Solution: Split into 60-record chunks to keep workflow execution lightweight.

5. **Timestamped Naming** — Static filenames (`clutch-leads-1.csv`) created Google Drive collisions and version confusion. Solution: Add ISO timestamp to filename (`clutch-leads-YYYY-MM-DDTHH-MM-1.csv`) for chronological tracking.

## Repository Scope
This repository serves as a project showcase and documentation hub. 
