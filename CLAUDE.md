# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Running Tests
```bash
# Run all tests
PYCHESS_UNITTEST=true PYTHONPATH=lib python3 -m unittest discover -s testing -p "*.py" -v

# Run a single test file
PYCHESS_UNITTEST=true PYTHONPATH=lib python3 -m unittest testing.TestBoardTransforms -v

# Using the convenience script
./run_tests.sh
```

### Linting & Formatting
```bash
# Run all pre-commit hooks
pre-commit run --all-files

# Individual tools
black lib/
flake8 lib/
pyupgrade --py310-plus lib/
```

### Running the Application
```bash
python pychess
```

### Build Artifacts (required before running tests in CI)
```bash
# Generate ECO database
PYTHONPATH=lib python3 lib/pychess/Database/pgn2ecodb.py

# Generate theme previews
PYTHONPATH=lib python3 create_theme_preview.py
```

## Architecture

PyChess is a GTK-based chess client built with Python 3, PyGObject, and asyncio. The event loop integrates GLib (optionally via `gbulb`) with Python asyncio.

### Entry Points
- `pychess` — main executable; validates dependencies, initializes GTK, starts the app
- `lib/pychess/Main.py` — `PyChess` GTK Application class (implements `GladeHandlers`)
- `lib/pychess/__init__.py` — defines `VERSION` and `MSYS2` flag

### Major Subsystems

**`lib/pychess/perspectives/`** — Modular UI sections (Welcome, Games, FICS, Database, Learn). Each perspective is a self-contained panel that users can switch between.

**`lib/pychess/Players/`** — Engine interfaces. `UCIEngine.py` and `CECPEngine.py` wrap external engines; `PyChess.py` is the built-in engine. `engineNest.py` handles discovery and `engineList.py` manages configuration.

**`lib/pychess/Utils/`** — Core game logic: `GameModel.py` manages game state; `Board.py` and `Move.py` handle board representation and move generation; `eval.py` does position evaluation.

**`lib/pychess/Variants/`** — 30+ chess variant implementations (Fischer Random, Crazyhouse, Atomic, etc.), each subclassing the core board/rules.

**`lib/pychess/Savers/`** — File format handlers: `pgn.py` (primary), `fen.py`, `epd.py`, `html.py`, `database.py`. Used for import/export.

**`lib/pychess/ic/`** — Internet Chess Server support: `FICSConnection.py` and `FICSObjects.py` for FICS; `icc.py` for ICC; `ICGameModel.py` wraps game state for network games; `TimeSeal.py` implements the FICS timing protocol.

**`lib/pychess/widgets/`** — UI components. `BoardView.py` (~86KB) renders the board via Cairo; `BoardControl.py` (~49KB) handles input and drag-and-drop; `gamewidget.py` manages the overall game panel.

**`lib/pychess/System/`** — Infrastructure: `conf.py` (configuration), `Log.py`/`LogEmitter.py` (logging), `prefix.py` (path resolution), `uistuff.py` (GTK utilities).

### Key Conventions
- `PYTHONPATH=lib` must be set when running any code; the package lives under `lib/pychess/`.
- UI layouts are in `glade/` as Glade XML files; loaded at runtime by GTK.
- `PYCHESS_UNITTEST=true` disables parts of the app that require a display during tests.
- Flake8 ignores: E203, E402, E501, F403, F405, F824 (see `setup.cfg`).
- Python 3.10+ required; code style enforced by black with `--target-version py310`.
