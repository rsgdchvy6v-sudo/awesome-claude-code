# CLAUDE.md

This file provides guidance to Claude Code when working with the awesome-claude-code repository.

## Project Overview

Awesome Claude Code is a highly automated curated list of resources for Claude Code users — slash-commands, CLAUDE.md files, CLI tools, hooks, skills, and more. The project's core design is **automation-first**: a single CSV file drives all README generation, and GitHub Actions handle resource submissions end-to-end.

- **License**: CC-BY-NC-ND 4.0 (Creative Commons Non-Commercial, No Derivatives)
- **Python**: 3.11+
- **Source of truth**: `THE_RESOURCES_TABLE.csv` (226+ resources, 20 columns)

## Repository Layout

```
awesome-claude-code/
├── THE_RESOURCES_TABLE.csv      # Single source of truth for all resources
├── acc-config.yaml              # README generation config (root_style, styles)
├── Makefile                     # All dev commands (50+ targets)
├── pyproject.toml               # Python deps and tooling config
├── .pre-commit-config.yaml      # Pre-commit hooks (ruff, tests, readme-sync)
├── README.md                    # Generated root README (style set in acc-config.yaml)
├── README_ALTERNATIVES/         # Generated alternative README styles (44+ files)
├── assets/                      # SVG badges and visual assets (800+ files)
├── data/                        # Repository ticker data (CSV)
├── docs/                        # Documentation
│   ├── CONTRIBUTING.md          # Submission guidelines
│   ├── HOW_IT_WORKS.md          # Label system and submission flow
│   ├── README-GENERATION.md     # Full multi-list architecture docs
│   ├── COOLDOWN.md              # Enforcement/ban protocol
│   └── development/             # Dev-specific docs
├── scripts/                     # Python automation (13 subdirectories)
│   ├── readme/                  # README generation engine
│   │   └── generators/          # VisualReadmeGenerator, MinimalReadmeGenerator, etc.
│   ├── validation/              # Link validation, duplicate detection
│   ├── badges/                  # SVG badge generation
│   ├── categories/              # Category management
│   ├── ids/                     # Resource ID generation
│   ├── resources/               # Resource sorting and management
│   ├── testing/                 # Test utilities and regeneration cycles
│   ├── ticker/                  # GitHub stats ticker fetching
│   ├── graphics/                # SVG template rendering
│   ├── maintenance/             # Maintenance scripts
│   └── archive/                 # Archived scripts (excluded from linting)
├── templates/                   # README templates and config
│   ├── categories.yaml          # Category/subcategory definitions (IDs, icons, etc.)
│   ├── resource-overrides.yaml  # Per-resource overrides
│   ├── announcements.yaml       # Announcements (rendered in READMEs)
│   └── *.template.md            # Jinja-style README templates per style
├── tests/                       # pytest test suite (22 files)
├── resources/                   # Organized resource directories by type
└── tools/                       # Utility tools (readme_tree)
```

## Key Commands

Always use `venv/bin/python3` locally (CI uses `python3`). The Makefile handles this automatically.

```bash
# Setup
python3 -m venv venv && pip install -e ".[dev]"
pre-commit install

# README generation
make generate               # Regenerate README.md and all alternatives from CSV + SVG badges
make generate-toc-assets    # Regenerate subcategory TOC SVGs (after adding subcategories)

# Testing
make test                   # Run pytest suite
make coverage               # pytest with coverage (HTML + XML reports)
make mypy                   # Type checking
make ci                     # format-check + mypy + test (what CI runs)

# Code quality
make format                 # Ruff lint fix + format
make format-check           # Check only (no fix)

# Validation
make validate               # Validate all links in CSV
make validate-single URL=https://... # Validate a single resource URL
make validate-toc           # Validate TOC anchors against GitHub HTML

# Resource management
make sort                   # Sort CSV by category, sub-category, name
make generate-resource-id   # Interactive resource ID generator
make add-category           # Interactive category addition

# Regeneration tests
make test-regenerate        # Regenerate and fail if diff (strict)
make test-regenerate-allow-diff     # Same but allow diffs
make test-regenerate-cycle  # Full root/style-order regeneration cycle test

# Cleanup
make clean                  # Remove caches and test artifacts
make clean-all              # Also remove venv/
```

## Architecture: Multi-List Generation

The repository generates **four README styles** simultaneously from a single CSV source:

| Style | Generator Class | Output | Description |
|-------|----------------|--------|-------------|
| `extra` | `VisualReadmeGenerator` | `README_ALTERNATIVES/README_EXTRA.md` | Visual/themed with SVG assets |
| `classic` | `MinimalReadmeGenerator` | `README_ALTERNATIVES/README_CLASSIC.md` | Clean markdown, collapsible sections |
| `awesome` | `AwesomeReadmeGenerator` | `README_ALTERNATIVES/README_AWESOME.md` | Awesome-list compliant |
| `flat` | `ParameterizedFlatListGenerator` | `README_ALTERNATIVES/README_FLAT_*.md` (44 files) | Sortable/filterable tables |

The configured `root_style` in `acc-config.yaml` is also written to `README.md`. Currently set to `awesome`.

The flat view generates 44 combinations: 11 category filters × 4 sort types (A-Z, Updated, Created, Releases).

### Resource ID Format

IDs follow `{prefix}-{hash}` where hash is the first 8 chars of SHA256(display_name + primary_link):

```
skill-  → Agent Skills
cmd-    → Slash-Commands
wf-     → Workflows & Knowledge Guides
tool-   → Tooling
claude- → CLAUDE.md Files
hook-   → Hooks
doc-    → Official Documentation
```

## Data: THE_RESOURCES_TABLE.csv

This CSV is the **only file you should edit** when adding/modifying resources. Never hand-edit generated READMEs.

Key columns: `ID`, `Display Name`, `Category`, `Sub-Category`, `Primary Link`, `Secondary Link`, `Author Name`, `Author Link`, `Active`, `Date Added`, `Last Modified`, `Last Checked`, `License`, `Description`, `Removed From Origin`, `Stale`, `Repo Created`, `Latest Release`, `Release Version`, `Release Source`

After editing the CSV, always run `make generate` to sync all READMEs.

## Category System

Categories are defined in `templates/categories.yaml`. The 9 main categories:

1. **Agent Skills** (🤖) — `skill-` prefix
2. **Workflows & Knowledge Guides** (🧠) — `wf-` prefix
3. **Tooling** (🧰) — `tool-` prefix (IDE Integrations, Usage Monitors, Orchestrators, Config Managers)
4. **Status Lines** (📊)
5. **Hooks** (🪝) — `hook-` prefix
6. **Slash-Commands** (🔪) — `cmd-` prefix (8 subcategories)
7. **CLAUDE.md Files** (📂) — `claude-` prefix
8. **Alternative Clients** (📱)
9. **Official Documentation** (🏛️) — `doc-` prefix

To add a new category: `make add-category` (interactive) then run `make generate`.

## Submission Workflow (GitHub-Automated)

**Human submissions**: Use the GitHub web UI issue form only. Never PRs, never `gh` CLI.

**Bot-driven flow**:
1. User fills issue form → labeled `resource-submission`
2. Bot validates (URL, duplicates, license, format) → labels `validation-passed` or `validation-failed`
3. Maintainer reviews → `/approve`, `/request-changes`, or `/reject`
4. On `/approve` → bot creates branch `add-resource/category/name-timestamp`, adds to CSV, runs `make generate`, opens PR
5. PR merged → badge notification sent to resource's GitHub repo

**Label state machine**: `resource-submission` → `validation-passed`/`validation-failed` → (`changes-requested`) → `approved` + `pr-created`/`error-creating-pr`

## GitHub Workflows (`.github/workflows/`)

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ci.yml` | Push/PR | format-check, mypy, tests |
| `submission-enforcement-v2.yml` | Issue events | Main resource submission handler |
| `handle-resource-submission-commands.yml` | Issue comments | `/approve`, `/reject`, `/request-changes` |
| `validate-links.yml` | Scheduled | Periodic link health check |
| `check-repo-health.yml` | Scheduled | Overall repo health monitoring |
| `update-repo-ticker.yml` | Scheduled | Fetch GitHub stats ticker data |
| `notify-on-merge.yml` | PR merge | Badge notifications to resource repos |
| `close-resource-pr.yml` | PR events | Auto-close stale resource PRs |

## Development Conventions

### Python Style

- **Ruff** for linting and formatting (line length 100, Python 3.11 target)
- **mypy** for type checking
- Imports sorted by isort (via Ruff)
- `scripts/archive/` is excluded from linting

### Pre-commit Hooks

Pre-commit runs automatically on `git commit`:
1. Large file check, merge conflict detection, YAML/JSON validation
2. Ruff lint (with `--fix`) and format
3. `make test` (full pytest suite)
4. Check that README matches CSV (fails if `make generate` would change anything)

**This means every commit must have a passing test suite and a synced README.** If you edit the CSV, run `make generate` before committing.

### Adding a New README Style

1. Create generator class extending `ReadmeGenerator` in `scripts/readme/generators/`
2. Create template in `templates/README_NEWSTYLE.template.md` (must include `{{STYLE_SELECTOR}}`)
3. Register in `STYLE_GENERATORS` in `scripts/readme/generate_readme.py`
4. Create style badge SVG in `assets/badge-style-newstyle.svg`
5. Add entry to `styles:` and `style_order:` in `acc-config.yaml`
6. Run `make generate`

### Adding/Removing Flat List Categories or Sort Types

See `scripts/readme/generators/flat.py` — update `FLAT_CATEGORIES` or `FLAT_SORT_TYPES` and run `make generate`. Removing a category also requires manually deleting orphaned `.md` files from `README_ALTERNATIVES/`.

## Git Workflow

- Never push directly to `main`
- Branch naming: `feat/...`, `fix/...`, `add-resource/category/name-timestamp` (bot-generated)
- Always run `make ci` before pushing
- PRs only for bug fixes, docs improvements, or tooling changes — NOT for resource submissions

## Testing

```bash
make test        # Full suite (22 test files)
make coverage    # With HTML coverage report in htmlcov/
```

Key test files in `tests/`:
- `test_generate_readme.py` — README generation correctness
- `test_flat_list_generator.py` — Flat list variants
- `test_validate_links.py` — Link validation
- `test_sort_resources.py` — Resource sorting
- `test_toc_anchor_validation.py` — TOC anchor integrity
- `test_category_utils.py` — Category utilities

## Environment Variables

```bash
GITHUB_TOKEN    # Required for GitHub API calls (avoid rate limiting)
CI=true         # Set by GitHub Actions (switches to system python3)
```

## Assets

SVG assets in `assets/` are generated — do not edit them manually:
- `badge-style-*.svg` — Style selector badges (Extra, Classic, Awesome, Flat)
- `badge-sort-*.svg` — Sort type badges (A-Z, Updated, Created, Releases)
- `badge-cat-*.svg` — Category filter badges
- `badge-*.svg` — Per-resource themed initials badges

Regenerated by `make generate` or `make generate-toc-assets`.
