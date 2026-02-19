# Anki Sync

Bidirectional vocabulary synchronization between local YAML files and Anki Desktop via Anki-Connect.

## Prerequisites

- Anki Desktop running with the [Anki-Connect](https://ankiweb.net/shared/info/2055492159) add-on (code: `2055492159`)

## Installation

```bash
pipx install git+https://github.com/astavonin/anki-sync.git
```

**Development:**

```bash
git clone git@github.com:astavonin/anki-sync.git
pipx install -e ./anki-sync
```

## Usage

```bash
anki-sync setup          # create deck and note model in Anki
anki-sync push           # push local YAML → Anki
anki-sync pull           # pull Anki edits → local YAML
anki-sync diff           # preview pending changes
anki-sync status         # show sync state
anki-sync verify         # check sync integrity
```

Options available on all commands: `--config <path>`, `--dry-run`, `--verbose`.

See `CLAUDE.md` for configuration reference, YAML format, and troubleshooting.
