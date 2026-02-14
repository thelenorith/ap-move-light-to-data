# ap-move-light-to-data

[![Test](https://github.com/jewzaam/ap-move-light-to-data/workflows/Test/badge.svg)](https://github.com/jewzaam/ap-move-light-to-data/actions/workflows/test.yml)
[![Coverage](https://github.com/jewzaam/ap-move-light-to-data/workflows/Coverage%20Check/badge.svg)](https://github.com/jewzaam/ap-move-light-to-data/actions/workflows/coverage.yml)
[![Lint](https://github.com/jewzaam/ap-move-light-to-data/workflows/Lint/badge.svg)](https://github.com/jewzaam/ap-move-light-to-data/actions/workflows/lint.yml)
[![Format](https://github.com/jewzaam/ap-move-light-to-data/workflows/Format%20Check/badge.svg)](https://github.com/jewzaam/ap-move-light-to-data/actions/workflows/format.yml)
[![Type Check](https://github.com/jewzaam/ap-move-light-to-data/workflows/Type%20Check/badge.svg)](https://github.com/jewzaam/ap-move-light-to-data/actions/workflows/typecheck.yml)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

Move complete directory trees containing light frames and their calibration to the data directory atomically.

## Documentation

This tool is part of the astrophotography pipeline. For comprehensive documentation including workflow guides and integration with other tools, see:

- **[Pipeline Overview](https://github.com/jewzaam/ap-base/blob/main/docs/index.md)** - Full pipeline documentation
- **[Workflow Guide](https://github.com/jewzaam/ap-base/blob/main/docs/workflow.md)** - Detailed workflow with diagrams
- **[ap-move-light-to-data Guide](https://github.com/jewzaam/ap-base/blob/main/docs/tools/ap-move-light-to-data.md)** - Detailed usage guide for this tool

## Overview

Automates the workflow step between blinking/reviewing light frames and processing them. The tool moves complete directory trees atomically when all lights have required calibration and all calibration is self-contained within the tree.

**Key features:**
- Moves entire directory trees atomically (prevents shared calibration file issues)
- Requires calibration to be self-contained (inside the tree being moved)
- Clear indicators: moved directories are ready, remaining directories need more calibration
- Supports multi-session workflows (move complete sessions, leave incomplete ones)
- Searches for calibration in light directory and parent directories within the tree

## Installation

### Development

```bash
make install-dev
```

### From Git

```bash
pip install git+https://github.com/jewzaam/ap-move-light-to-data.git
```

## Usage

```bash
python -m ap_move_light_to_data <source_dir> <dest_dir> [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--debug`, `-d` | Enable debug output |
| `--dryrun`, `-n` | Show what would be done without moving files |
| `--quiet`, `-q` | Suppress progress output |
| `--scale-dark` | Scale dark frames using bias compensation (allows shorter exposures). Default: exact exposure match only |
| `--path-pattern REGEX` | Filter directories by regex pattern |

### Examples

```bash
# Move complete directory trees from 10_Blink to 20_Data
python -m ap_move_light_to_data 10_Blink 20_Data

# Dry run to preview what would be moved
python -m ap_move_light_to_data 10_Blink 20_Data --dryrun

# Process only accept directories (skip rejected)
python -m ap_move_light_to_data 10_Blink 20_Data --path-pattern ".*accept.*"

# Enable bias-compensated dark scaling (allows shorter dark exposures)
python -m ap_move_light_to_data 10_Blink 20_Data --scale-dark
```

## How It Works

The tool recursively evaluates directory trees and moves them atomically when:
1. All light frames have required calibration (darks, flats, bias if needed)
2. All calibration files are self-contained within the tree (not in parent directories)

**Calibration matching criteria:**
- **Darks**: Camera, set temperature, gain, offset, readout mode, exposure (unless `--scale-dark`)
- **Flats**: Camera, set temperature, gain, offset, readout mode, filter
- **Bias**: Only required with `--scale-dark` when dark exposure doesn't match light exposure

**Recursive evaluation:**
- Complete TARGET directory → moves entire TARGET atomically
- Incomplete TARGET but complete DATE directories → moves only complete DATEs
- Incomplete directories are skipped and reported with missing calibration details

**Example:** If `M31/DATE_2026-02-07` is complete but `M31/DATE_2026-02-08` is missing darks, only the complete date moves. The incomplete date remains as a clear indicator that more calibration is needed.
