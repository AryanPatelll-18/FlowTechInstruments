# `batch_download.py`

## Purpose
Downloads a large list of `tmp.link` files sequentially by calling the backend API first and then fetching each file directly.

## What It Does
- Defines the same large `URLS` list.
- Creates `/mnt/agents/output/app`.
- For each URL:
  - Extracts the `ukey` from the URL.
  - Sends a POST request to `https://connect.cntmp.link/api_v2/file` with `action=download_req`.
  - Reads the returned direct download URL.
  - Derives the final filename from that direct URL.
  - Downloads the file with `requests.get(..., stream=True)`.
  - Writes the file to disk in chunks.
- Tracks success and failure counts.
- Sleeps briefly between downloads to reduce request pressure.

## Main Dependencies
- `requests`
- `os`
- `time`

## Execution Flow
1. Ensure the output folder exists.
2. Loop through URLs one by one.
3. Request a temporary direct file URL from the API.
4. Download the file content from that direct URL.
5. Store the file in the output directory.
6. Print a final success/failure summary.

## Strengths
- Simple and easy to follow.
- Streams file content instead of loading everything into memory at once.
- Avoids browser automation.

## Risks / Limitations
- Fully sequential, so it will be slow on large lists.
- No resume logic if the script is interrupted.
- Does not skip already downloaded files.
- Relies on the API response format and the direct URL being valid.

