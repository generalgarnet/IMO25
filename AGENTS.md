# Repository Guidelines

## Project Structure & Module Organization
The core agents live in `code/`: `agent.py` (Gemini), `agent_oai.py` (OpenAI), and `agent_xai.py` (XAI). Parallel orchestration sits in `code/run_parallel.py`, while `code/res2md.py` extracts the last JSON result from log files. Reference problem statements are stored in `problems/`, and sample execution traces reside under `run_logs*/`. Place any experimental agent variants in `code/community_codes/` and document them in their module docstrings.

## Build, Test, and Development Commands
Create a virtual environment and install dependencies with `python -m venv .venv` followed by `.venv\Scripts\Activate` and `pip install -r requirements.txt`. Run a single agent from the repo root via `python code/agent.py problems/imo2025_p1.txt --log logs/p1_gemini.log`. Launch multi-agent runs with `python code/run_parallel.py problems/imo2025_p1.txt -n 10 -t 300 -d logs/p1_batch`. Use `python code/res2md.py logs/results.jsonl` to inspect the final JSON payload from a batch run.

## Coding Style & Naming Conventions
Follow PEP 8 with four-space indentation and descriptive `snake_case` identifiers; reserve ALL_CAPS for constants such as `MODEL_NAME`. Keep modules self-contained with top-level configuration blocks and helper functions grouped by responsibility. Provide triple-quoted docstrings for public functions and concise inline comments only where control flow is non-obvious. Prefer structured logging through the existing `log_print` hook and guard outbound HTTP calls made with `requests` behind dedicated helper functions for easier retries and error handling.

## Testing Guidelines
Automated tests are not yet present; add new ones under `tests/` using `pytest` or `unittest` naming files `test_<module>.py`. Ensure each agent change is exercised with a smoke run such as `python code/agent.py problems/imo2025_p1.txt --log tmp.log` and verify that `run_parallel.py` still spins up the requested number of workers. Capture reproducible scenarios by committing sanitized log excerpts in `run_logs/notes.md` when fixing regressions.

## Commit & Pull Request Guidelines
Recent history favors short, imperative subject lines (e.g., `Add agents for OAI and XAI`). Keep commits focused on a single concern and include updates to documentation or sample prompts when behavior changes. Pull requests should summarize intent, list test commands executed, link any tracked issue, and attach relevant log snippets or screenshots demonstrating successful runs.

## Security & Configuration Tips
Never hard-code API keys; rely on the environment variables `GOOGLE_API_KEY`, `OPENAI_API_KEY`, and `XAI_API_KEY`. Redact tokens and sensitive problem statements from shared logs. When contributing new providers, isolate credentials in a dedicated configuration helper and update `.gitignore` if additional secret files are introduced.
