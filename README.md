# Take-Home Exercise: SEC Filings Data Pipeline

**Role:** Software Developer (Python) – Web Scraping \& Data Extraction
**Time limit:** 4 hours maximum (please stop at 4 hours and record what remains)
**Submission:** GitHub repository (public, or private with access granted to \[REVIEWER GITHUB HANDLES]) by \[2 days before interview]
**Follow-up:** 30-minute technical conversation about your code and design

\---

## 1\. Overview

This exercise describes a complete, production-style pipeline that collects investment data from SEC EDGAR and turns it into analysis-ready datasets. It is representative of the work this team does.

**You will build only a core slice of it.** Every requirement below is tagged with a scope:

1. **BUILD** – implement it within the 4 hours.
2. **DISCUSS** – do not implement it. Read it and come prepared to talk through how you would approach it in the interview. No written answer is needed.

We are assessing how you design for **reliability, data quality, and maintainability**. A small slice that is solid, tested, and honestly documented is exactly what we want. Please do not exceed 4 hours. Extra features will not earn extra credit.

You may use any resources, including AI coding assistants. Please disclose where you used them. In the interview, you should be able to explain and defend any line in the repository.

\---

## 2\. The Full Pipeline at a Glance

1. **Collect** quarterly 13F holdings for institutional managers from EDGAR's REST API, HTML index pages, and XML documents. *(Part A)*
2. **Collect** recent 8-K filings from EDGAR's JavaScript-rendered full-text search. *(Part B)*
3. **Store** the raw responses in object storage and the curated data in a SQL database, with validation and lineage. *(Part C)*
4. **Operate** the pipeline on a schedule in the cloud, with monitoring and alerting. *(Part D)*
5. **Engineer** it to a team standard, with tests and CI. *(Part E)*

\---

## 3\. Rules of Engagement (Mandatory)

1. **Sources.** Use only the SEC's own domains (sec.gov and its subdomains).
2. **SEC fair-access policy.**

   1. Send a declared User-Agent that includes your name and email address.
   2. Stay well under 10 requests per second.
   3. Cache responses so that re-runs do not re-download content.
3. **Compliance.** Honor robots.txt, and do not attempt to bypass CAPTCHAs, blocks, or access controls.
4. **Secrets.** Do not commit credentials, keys, or personal tokens.

\---

## 4\. Part A – 13F Holdings Collection

|ID|Scope|Requirement|
|-|-|-|
|A.1|BUILD|**Configuration.** Read a configurable list of managers by CIK. Use Berkshire Hathaway (CIK 0001067983) plus one or two others of your choice.|
|A.2|BUILD|**Discovery (REST API).** Use EDGAR's JSON submissions API to find each manager's 13F-HR filings for report periods from **Q3 2022 through the most recent available**.|
|A.3|BUILD|**Extraction (HTML + XML).** From each filing's index page, locate and download the information table XML. Parse it into holdings records.|
|A.4|DISCUSS|**Amendments.** 13F-HR/A filings come in more than one type. How would you apply them, and how would you restate history?|
|A.5|BUILD|**Idempotency.** Re-running the pipeline must not duplicate data or re-download filings.|
|A.6|BUILD|**Fetch layer.** Make requests concurrently (asyncio or threads) under a single global rate limit, with retries, backoff, and timeouts.|

\---

## 5\. Part B – JavaScript-Rendered Source

|ID|Scope|Requirement|
|-|-|-|
|B.1|DISCUSS|EDGAR Full-Text Search (https://www.sec.gov/edgar/search/) has a JavaScript front end. How would you collect 8-K filings matching a list of keywords over the last 30 days?|
|B.2|DISCUSS|Would you drive the page with a headless browser (Playwright or Selenium) or call the data source it uses behind the scenes? What are the trade-offs? Feel free to explore the page in your browser's developer tools beforehand.|
|B.3|DISCUSS|How would you handle pagination and de-duplication across keywords?|

\---

## 6\. Part C – Data Quality, Storage, and Analytics

|ID|Scope|Requirement|
|-|-|-|
|C.1|BUILD (light)|**Raw zone.** Save every fetched response unchanged to a local directory that stands in for object storage. Record the URL, timestamp, HTTP status, and content hash for each. *(DISCUSS: how this maps to S3, Azure Blob, or GCS.)*|
|C.2|BUILD|**Curated zone.** Load normalized holdings into a SQL database. PostgreSQL is preferred; SQLite is acceptable to save time.|
|C.3|BUILD|**Comparability.** Monetary values must be in consistent units and comparable across **every** quarter in scope.|
|C.4|BUILD (partial)|**Validation.** Quarantine failing records rather than dropping them silently. Check that required fields are present and that there are no duplicate positions within a filing. *(DISCUSS: reconciling holdings against the totals reported in the filing.)*|
|C.5|BUILD|**Lineage.** Every curated row must be traceable back to its source filing and raw file.|
|C.6|BUILD|**Analytics.** Provide a SQL query or view that lists, per manager and quarter, the new, exited, increased, and decreased positions.|

\---

## 7\. Part D – Operations and Cloud

|ID|Scope|Requirement|
|-|-|-|
|D.1|BUILD|**Entry point.** Provide a single CLI command that runs the pipeline.|
|D.2|DISCUSS|How would you containerize and run the pipeline locally end to end (for example, with Docker Compose)?|
|D.3|DISCUSS|What would you log per run, and how would you make scraper health visible to the team?|
|D.4|BUILD|**Structure-change detection.** If a parser returns zero or implausible results, for example because a page layout changed, the run must **fail loudly** with a non-zero exit code and a clear message. It must not write empty or partial data.|
|D.5|DISCUSS|How would you schedule this, and what freshness alert would you set for quarterly filings?|
|D.6|DISCUSS|How would you deploy this on AWS, Azure, or GCP using infrastructure as code? Which services would you choose and why?|

\---

## 8\. Part E – Engineering Quality

|ID|Scope|Requirement|
|-|-|-|
|E.1|BUILD|Use Python 3.11+ with type hints and pinned dependencies.|
|E.2|BUILD|Write pytest tests that run **fully offline** using saved fixtures. Cover the parser and the C.3 normalization, and include one test that simulates a changed document structure (D.4).|
|E.3|BUILD|Set up a minimal GitHub Actions workflow that runs the tests on every push.|
|E.4|BUILD|Keep an incremental commit history; please do not squash into a single commit.|

\---

## 9\. README Requirements (Keep It Short)

Your README must include:

1. Setup and run instructions.
2. Your key design decisions and trade-offs, in a few bullets.
3. A **status table** of the BUILD items, each marked Done, Partial, or Not done.
4. An approximate time log.
5. An AI-assistance disclosure: which tools you used and for what.

\---

## 10\. Suggested Time Budget

|Activity|Hours|
|-|-|
|Setup and design|0.5|
|Part A – collection and fetch layer|1.5|
|Part C – normalization, storage, validation, analytics|1.0|
|D.1, D.4, and Part E – CLI, failure handling, tests, CI|0.75|
|README|0.25|
|**Total**|**4.0**|

If you run short on time, prioritize A.2, A.3, C.3, D.4, and E.2. Mark anything unfinished in the status table. Honest gaps are fine.

\---

## 11\. The Follow-Up Conversation (30 Minutes)

1. **About 5 minutes:** you walk us through your code and your most important design decision.
2. **About 20 minutes:** we discuss your code and two or three of the DISCUSS items.
3. **About 5 minutes:** your questions for us.

Good luck. We are looking forward to seeing how you think.

# PythonWebScraping