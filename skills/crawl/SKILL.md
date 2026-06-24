---
name: crawl
description: Crawl multiple pages of a website and return their content as clean markdown. Use when the user wants more than one page — "crawl this site", "get all the docs pages", "extract the whole section". Discovers links from a seed page and fetches them through UpRock's network.
---

# UpRock: Crawl a site

Crawl a set of pages starting from a seed URL and return their content as markdown. UpRock's MCP exposes per-page fetching; this skill drives a controlled breadth-first crawl on top of it.

## How to do it

1. Clarify scope if not given: how many pages (default cap ~20), and which paths to stay within (default: same host and, if the seed is like `example.com/docs/...`, the same path prefix).

2. **Seed fetch:** call **`crawl_fetch`** on the seed URL (`CRAWL_FULL_PAGE`), then **`resource_fetch`** the `crawl://…/html` resource so you can read links. (Use HTML here, not markdown, so anchors/hrefs are visible.)

3. **Extract links** from the seed HTML. Keep only links that:
   - are on the same host (and same path prefix, if one was implied), and
   - are not already visited, not anchors/`mailto:`/`tel:`/asset files (`.png`, `.pdf`, `.zip`, …).
   Deduplicate. This is the frontier.

4. **Fetch the frontier:** for each URL up to the page cap, call `crawl_fetch` (`CRAWL_FULL_PAGE`), then `resource_fetch` the `…/markdown` resource. These run independently — issue them in batches.

5. (Optional depth > 1) Extract new in-scope links from fetched pages and repeat until the page cap or requested depth is reached. Always respect the cap so credits are not burned unexpectedly.

6. Return a structured result: a list of `{ url, title, markdown }`, or write each page to a file under `.uprock/crawl/` if the user prefers files over inline output (keeps the context window clean for large crawls).

## Notes
- Honor the page cap and scope strictly; confirm before crawling more than ~50 pages.
- Respect `robots.txt` intent — do not crawl pages the user has no right to access.
- For "search the web and scrape results" (not one site), use `/uprock:search`. For autonomous, goal-driven gathering across many sites, use `/uprock:agent`.
