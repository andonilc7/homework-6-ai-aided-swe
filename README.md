# Architectural Verifier Agent

This Python CLI reviews Git changes against a target repository's `ARCHITECTURE.md`. It makes one model call per review and prints a verdict with evidence, corrections, or questions for the developer. It reads the target files and writes a local execution log.

## Setup

Use Python 3.10 or newer, Git, and Ollama. Run these commands from the Homework 6 repository root.

```sh
python3 -m venv /tmp/architecture-verifier-venv
source /tmp/architecture-verifier-venv/bin/activate
python -m pip install -r requirements.txt
```

Open the Ollama app or run `ollama serve` in another terminal. Download the default model with `ollama pull qwen3` if it is missing from `ollama list`. The script calls `ollama.chat()` with a 2,000-token output budget. Pass `--model qwen3:4b` to use that tag if you have it installed. [Qwen3 on Ollama](https://ollama.com/library/qwen3)

## Run a review

The included `test_repo/` is a folder inside this Git repository. Run the following command from the Homework 6 folder to review the sample. The expected verdict is `FAIL` because `pricing.py` deliberately calls `print()`.

```sh
python run_agent.py \
  --target ./test_repo --baseline --log-file ./logs/baseline.log
```

`--baseline` compares the selected files with an empty tree so a clean starting repository can be reviewed. A run without `--baseline` compares the working tree to `HEAD` and includes staged edits, unstaged edits, and non-ignored new files. Commit the starting files before reviewing edits without `--baseline`. Use `--base <commit>` to review against a specific commit.

```sh
python run_agent.py \
  --target ./test_repo --log-file ./logs/iterative.log
```

Pass a Git repository or a folder within one as `--target`. The agent reads `ARCHITECTURE.md` from that folder and scopes the diff to it. Use `--path` to narrow the diff further and `--context` to include a related file. Both options accept paths relative to the target folder and can be repeated.

Add `--dry-run` to see the exact repository content that would be sent to Ollama, without calling the model or writing a log. The CLI stops if the review input exceeds 12,000 bytes to leave room for the prompt and response in the local context window. Narrow the scope and rerun if needed.

## Interpret the result

| Result | Exit code | Meaning |
| --- | --- | --- |
| `PASS` | 0 | The supplied evidence supports compliance. |
| `FAIL` | 1 | The report cites a documented rule and code evidence for each violation. |
| `UNCERTAIN` | 3 | The report asks for the rules or files needed to decide. |
| `NO_CHANGES` | 0 | Git found no changes in scope, so the CLI skipped the model call. |
| `ERROR` | 2 | Git, input collection, the model, or logging failed. |

The log defaults to `architecture-verifier.log` and records one JSON line per run, including the scope, duration, verdict or error, and token usage when available. Reports can quote source lines, so review the log before sharing it.

For the assignment's iterative run, repeat the baseline command against the same files or rerun your chosen scope after a partial fix. Use a separate log to compare verdicts, cited evidence, elapsed time, and token counts.
