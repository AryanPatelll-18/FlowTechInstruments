# `download_tmp.py`

## Purpose
Automates downloading a long list of `tmp.link` files by using Playwright to drive the website in a headless Chromium browser.

## What It Does
- Defines a large `URLS` list of `tmp.link/f/...` links.
- Creates `/mnt/agents/output/app` if it does not already exist.
- Launches Chromium in headless mode through `playwright.async_api`.
- For each URL:
  - Converts the file link from `/f/<ukey>` to `/link/file?ukey=<ukey>`.
  - Opens the page and waits for the page title to load.
  - Derives a filename from the page title, or falls back to the `ukey` value.
  - Clicks the download button labeled `下载`.
  - Saves the downloaded file into the download directory.
- Logs progress and catches per-file errors so the loop continues.

## Main Dependencies
- `asyncio`
- `playwright.async_api.async_playwright`
- `os`

## Execution Flow
1. Start a headless browser.
2. Reuse one browser context and one page for all URLs.
3. Visit each tmp.link file page.
4. Wait for the download event after clicking the button.
5. Save the file locally with the chosen filename.
6. Continue even if a single item fails.

## Strengths
- Uses the real browser workflow, which may work when direct HTTP download endpoints are not easy to access.
- Handles site behavior that depends on client-side rendering.
- Keeps going after failures.

## Risks / Limitations
- Much slower than direct HTTP downloading.
- Depends on page structure and the Chinese download button text.
- Filename extraction is heuristic and may not always match the real file name.
- Requires Playwright and a Chromium browser to be installed.

