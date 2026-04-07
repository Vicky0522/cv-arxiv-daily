# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an automated arXiv paper tracker that fetches Computer Vision and ML papers daily, organizes them by keywords, and publishes to README.md and GitHub Pages. Everything runs via GitHub Actions on a schedule.

## Commands

**Install dependencies:**
```bash
pip install -r requirements.txt
```

**Run the daily paper fetch manually:**
```bash
python daily_arxiv.py
```

**Run the weekly code link updater:**
```bash
python daily_arxiv.py --update_paper_links
```

## Architecture

### Data Flow

1. `config.yaml` defines keywords and filter terms (e.g., `"diffusion"` with filters `["diffusion", "score matching"]`)
2. `daily_arxiv.py` queries the arXiv API for each keyword, fetches code links from `arxiv.paperswithcode.com`, and optionally falls back to GitHub search
3. New papers are merged into JSON database files (`docs/*.json`) — these act as persistent storage and prevent duplicates
4. JSON is converted to Markdown and written to `README.md`, `docs/index.md`, and optionally `docs/wechat.md`
5. GitHub Actions commits and pushes all changed files automatically

### Two Workflows

- `.github/workflows/cv-arxiv-daily.yml` — runs every 12 hours, fetches new papers
- `.github/workflows/update_paper_links.yml` — runs every Monday, backfills missing code links for existing papers

### Key Script Functions (`daily_arxiv.py`)

- `demo(**config)` — main entry point, orchestrates the full pipeline
- `get_daily_papers(topic, query, max_results)` — queries arXiv API
- `get_code_link(qword)` — fetches code repo from paperswithcode API
- `update_json_file(filename, data_dict)` — merges new papers into JSON DB
- `json_to_md(filename, md_filename, ...)` — converts JSON to formatted Markdown
- `update_paper_links(filename)` — weekly job to fill in missing code links

### Configuration (`config.yaml`)

- Add new paper categories by adding entries under `keywords:` with a `filters:` list
- Toggle outputs with `publish_readme`, `publish_gitpage`, `publish_wechat`
- `max_results` controls how many papers per keyword per run
- JSON and Markdown output paths are all configurable

### JSON Storage Format

Papers are stored as pipe-delimited markdown table row strings, keyed by arXiv ID:
```json
{
  "diffusion": {
    "2306.14079": "|**2023-06-24**|**Title**|Author et.al.|[2306.14079](url)|**[link](github)**|\n"
  }
}
```
