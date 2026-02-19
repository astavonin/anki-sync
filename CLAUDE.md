# Anki Sync

Bidirectional vocabulary synchronization between local YAML files and Anki Desktop via Anki-Connect.

**Repo:** `~/projects/anki-sync`

---

## Installation

```bash
pipx install git+https://github.com/astavonin/anki-sync.git

# Development (editable)
pipx install -e ~/projects/anki-sync
```

## Commands

```bash
anki-sync setup          # create deck and note model in Anki
anki-sync push           # push local YAML → Anki
anki-sync pull           # pull Anki edits → local YAML
anki-sync diff           # preview pending changes
anki-sync status         # show sync state
anki-sync verify         # check sync integrity
```

Global options: `--config <path>`, `--dry-run`, `--verbose`

---

## Configuration

Search order (first found wins):
1. `--config` flag
2. `~/.config/anki_sync/config.yaml`
3. `./anki_sync_config.yaml`
4. Built-in defaults

```yaml
anki:
  host: "localhost"
  port: 8765
  timeout: 30
  deck_name: "Spanish::Vocabulary"
  model_name: "SpanishVocab"

sync:
  source_file: "~/.claude/memory/spanish-vocabulary.yaml"
  state_file: "~/.config/anki_sync/sync_state.json"
  conflict_log: "~/.config/anki_sync/conflicts.log"
  auto_sync_ankiweb: false

cards:
  include_examples: true
  include_notes: true
  max_example_length: 200
  tag_prefix: "anki_sync"
```

See `anki_sync/examples/config.yaml` for a full annotated example.

---

## Translation File Format

YAML list; `spanish` and `english` are required:

```yaml
- spanish: "sí"
  english: "yes"
  example: "Sí, funciona bien"   # optional
  notes: "Affirmative response"  # optional
  tags: ["common", "basic"]      # optional
```

**source_id** is the normalized (lowercase, stripped) Spanish term and must be unique across entries.

---

## Architecture

```
anki_sync/
├── cli.py                   # CLI entry point
├── config.py                # Config dataclass + file resolution
├── exceptions.py            # Exception hierarchy
├── client.py                # AnkiConnectClient (HTTP wrapper)
├── models/
│   ├── translation.py       # Translation dataclass + content hash
│   ├── card_mapping.py      # CardMapping dataclass
│   └── sync_state.py        # SyncState with atomic file saves
├── handlers/
│   ├── setup_handler.py     # Setup: create deck + model
│   ├── push_handler.py      # Push: local → Anki
│   └── pull_handler.py      # Pull: Anki → local
└── utils/
    └── logging_config.py    # Logging setup
```

**Key design decisions:**
- State saved atomically (write-to-temp + rename)
- State saved incrementally after each batch (minimizes data loss on crash)
- source_id collision detected before any sync begins
- Partial note failures are logged and skipped, not fatal
- Pull saves translations before updating state (prevents divergence)

---

## Development

```bash
# Run tests
pytest tests/ -v
pytest tests/ --cov=anki_sync --cov-report=term-missing

# Code quality
black anki_sync/
pylint anki_sync/
flake8 anki_sync/
mypy anki_sync/

# Or via Makefile
make test
make lint
make format
```

---

## Error Recovery

**Cannot connect to Anki-Connect:**
- Ensure Anki is running with Anki-Connect add-on installed (code: `2055492159`)

**Duplicate source_id:**
- Use distinct Spanish terms; normalization lowercases and strips whitespace

**Corrupted state file:**
```bash
cp ~/.config/anki_sync/sync_state.json{,.backup}
rm ~/.config/anki_sync/sync_state.json
anki-sync setup && anki-sync push
```
