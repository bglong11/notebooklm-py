# Using notebooklm-py in the Scientific Paper Writing Pipeline

## Setup (one-time)

```bash
cd /home/bglon_/github-ubuntu/notebooklm-py
sudo /usr/bin/python3.12 -m pip install --break-system-packages -e ".[browser]"
playwright install chromium
notebooklm login  # Opens browser — log in with Google account
```

Credentials are saved to `~/.notebooklm/storage_state.json`.

## Point the CLI at your existing notebook

```bash
# List notebooks to find your research one
notebooklm list

# Set it as active (copy the ID from the list)
notebooklm use <notebook-id>
```

Or set it as an env var to avoid running `use` each time:

```bash
export NOTEBOOKLM_NOTEBOOK_ID=<your-id>
```

## Query your notebook in the pipeline

```bash
# Ask questions and capture structured output with source citations
notebooklm ask "Summarize the key findings relevant to X" --json > research/summary.json

# Save answers directly as notes in the notebook
notebooklm ask "What methods are used in the literature?" --save-as-note --note-title "Methods Review"

# Get the full text of a specific source
notebooklm source list
notebooklm source fulltext <source-id> -o research/source_content.txt
```

## In the research step of the writing pipeline

```bash
notebooklm use <your-notebook-id>
notebooklm ask "What are the main arguments supporting my thesis on X?" --json
notebooklm ask "List contradicting evidence or limitations found in the sources" --json
notebooklm ask "Suggest a logical structure for a paper on X" --json
```

The `--json` flag returns structured output with source citations — pipe directly into downstream writing stages.

## Agent usage

An agent can call the CLI as a shell tool and parse the JSON response:

```python
import subprocess, json

def query_notebooklm(question: str) -> dict:
    result = subprocess.run(
        ["notebooklm", "ask", question, "--json"],
        capture_output=True, text=True
    )
    return json.loads(result.stdout)

response = query_notebooklm("What methods are used across the literature?")
```

| Agent task | Command |
|------------|---------|
| Ask research questions | `notebooklm ask "..." --json` |
| Save findings as notes | `notebooklm ask "..." --save-as-note --note-title "..."` |
| List sources | `notebooklm source list` |
| Get full source text | `notebooklm source fulltext <id> -o file.txt` |
| Generate briefing doc | `notebooklm generate report --format briefing-doc --wait` |

The `--json` flag returns the answer plus source citations, which the agent can use to ground its writing.

## Notes

- The CLI is stateless between calls; `use <id>` sets the active notebook in `~/.notebooklm/storage_state.json`.
- Use `--save-as-note` to persist important answers back into the notebook for reference.
- The notebook should already be populated via NotebookLM's deep search before running these steps.
- Authentication requires `~/.notebooklm/storage_state.json` from an initial `notebooklm login`. Agents run non-interactively as long as the session is valid. Re-run `notebooklm login` if it expires.
