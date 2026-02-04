# BULC-D Headless Lab Runner

> Run your Google Earth Engine science identically whether you're clicking "Run" in the browser or launching 100 experiments from your terminal.

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║  🧪  BULC-D HEADLESS LAB RUNNER v11.0                                         ║
║                                                                               ║
║  Browser ←→ Terminal parity for reproducible Earth Engine research            ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

## Overview

This system enables **Environment Switching** between:
1. **Browser Mode**: Google Earth Engine Code Editor
2. **Lab Mode**: Headless Node.js execution via `runner11.js`

The key insight is **structural mirroring**: your parameters exist in two forms (`.js` for the browser, `.json` for the lab), and a simple switch in your Caller script chooses between them.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           THE HEADLESS LAB SYSTEM                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────┐                      ┌─────────────────┐             │
│   │  BULCD-Params.js│                      │BULCD-Params.json│             │
│   │  (Browser)      │ ◄── Structural ──►   │  (Lab)          │             │
│   │                 │      Mirror          │                 │             │
│   └────────┬────────┘                      └────────┬────────┘             │
│            │                                        │                       │
│            │                                        │                       │
│            ▼                                        ▼                       │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                    BULCD-Caller-v51e.js                      │          │
│   │                                                              │          │
│   │   if (typeof inputParams !== 'undefined') {                  │          │
│   │     // LAB MODE                                              │          │
│   │     finalParams = inputParams;                               │          │
│   │   } else {                                                   │          │
│   │     // BROWSER MODE                                          │          │
│   │     finalParams = require('users/.../BULCD-Params');         │          │
│   │   }                                                          │          │
│   │                                                              │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                              │                                              │
│                              ▼                                              │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                       runner11.js                            │          │
│   │                    (The Lab Kitchen)                         │          │
│   │                                                              │          │
│   │   • Validates parameters                                     │          │
│   │   • Injects JSON as inputParams                              │          │
│   │   • Resolves GEE module paths                                │          │
│   │   • Provides UI shims                                        │          │
│   │   • Submits export tasks to GEE                              │          │
│   │                                                              │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Quick Start

### 1. Install Dependencies

```bash
npm install
```

### 2. Clone GEE Repositories

First, authenticate with Google Cloud:

```bash
# Install gcloud if not already installed, then:
gcloud auth login
```

Then clone the BULC-D repositories:

```bash
# Clone all default repositories (always pulls latest)
npm run setup

# Clone for a different user
node setup-gee-repos.js --user your_username

# Clone specific repos only
node setup-gee-repos.js --repos "r-2903-Dev,CommonCode"

# Just scan existing modules for missing dependencies
npm run setup:scan
```

**Note:** Running `npm run setup` always pulls the latest code from the repositories and refreshes the local `gee_modules/` directory.

### 3. Set Up GEE Authentication

Create a Google Cloud Service Account with Earth Engine access and download the JSON key:

```bash
# Place your key file in the project root
mv ~/Downloads/your-service-account-key.json ./service-account-key.json

# Or set environment variable
export GEE_KEY_PATH=/path/to/your/key.json
```

### 4. Run a Dry Test

```bash
# Validate without executing (no GEE calls)
npm test

# Or directly:
node runner11.js scripts_to_run/BULCD-Caller-v51e.js gee_modules experiments/BULCD-Params.json --dry-run
```

### 5. Run an Experiment

```bash
# Run with default parameters
npm run run:default

# Run with high thresholds
npm run run:high

# Run all experiments in batch
npm run batch
```

## Folder Structure

```
bulcd-runner/
│
├── runner11.js                    # The Lab Runner (Node.js)
├── service-account-key.json       # Your GEE service account key
├── package.json
│
├── scripts_to_run/                # Your GEE Caller scripts
│   └── BULCD-Caller-v51e.js       # Main caller with Environment Switch
│
├── experiments/                   # Experiment parameter files (JSON)
│   ├── BULCD-Params.json          # Default parameters
│   ├── high_threshold.json        # High threshold experiment
│   └── low_threshold.json         # Low threshold experiment
│
├── gee_modules/                   # Local copies of GEE modules
│   ├── CommonCode/                # Matches users/alemlakes/CommonCode
│   ├── BULC-D/                    # Matches users/alemlakes/BULC-D
│   │   └── BULCD-Params.js        # Master params for browser
│   └── BULC-Releases/             # Matches users/alemlakes/BULC-Releases
│       └── BULC-Module-107.js     # BULC-D core module
│
└── logs/                          # Execution logs (auto-created)
```

## Usage

### Basic Syntax

```bash
node runner11.js <caller.js> <modules_dir> <sidecar.json> [options]
```

### Arguments

| Argument | Description |
|----------|-------------|
| `caller.js` | Path to your GEE Caller script |
| `modules_dir` | Path to local GEE modules directory |
| `sidecar.json` | Path to experiment parameters JSON |

### Options

| Option | Description |
|--------|-------------|
| `--dry-run` | Validate without executing (no GEE calls) |
| `--verbose` | Enable verbose output |
| `--quiet` | Suppress GEE print() output (recommended for production) |
| `--silent` | Suppress all non-error output |
| `--no-log` | Disable file logging |
| `--batch` | Enable batch mode (sidecar.json is a directory) |
| `--parallel=N` | Run N experiments in parallel (batch mode) |
| `--help` | Show help message |

## Repository Setup

The `setup-gee-repos.js` script clones GEE repositories and prepares them for headless execution.

### Usage

```bash
node setup-gee-repos.js [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--user`, `-u` | GEE username (default: alemlakes) |
| `--repos`, `-r` | Comma-separated list of repos to clone |
| `--scan-only` | Only scan for dependencies, don't clone |
| `--verbose` | Enable verbose output |
| `--help` | Show help message |

### Behavior

- **Always pulls latest**: Each run fetches the latest code from the remote repositories
- **Clean refresh**: The `gee_modules/` directory is refreshed with each run
- **Auto .js extension**: GEE files without extensions get `.js` added automatically
- **Dependency scanning**: Detects missing dependencies from `require()` statements

### Default Repositories

The script clones these repositories by default:

| Repository | Description |
|------------|-------------|
| `r-2903-Dev` | Main BULC-D development repo |
| `r-2909-BULC-Releases` | BULC-D release modules |
| `r-2902-Dev` | Development support code |
| `r-2901-BULC-Dev` | BULC development modules |
| `CommonCode` | Shared utility functions |
| `CommonCode2` | Additional shared utilities |

### Examples

```bash
# Single experiment
node runner11.js scripts_to_run/BULCD-Caller.js gee_modules experiments/test.json

# Dry run (validation only)
node runner11.js scripts_to_run/BULCD-Caller.js gee_modules experiments/test.json --dry-run

# Verbose output
node runner11.js scripts_to_run/BULCD-Caller.js gee_modules experiments/test.json --verbose

# Batch execution (run all JSONs in a directory)
node runner11.js scripts_to_run/BULCD-Caller.js gee_modules experiments/ --batch
```

## Creating Experiments

### Parameter Structure

Each experiment JSON must have three sections:

```json
{
  "inputParameters": {
    "whichReduction": "nbr",
    "bandName_reduction": "nbr",
    "bandNameToFit": "nbr",
    "binCuts": [-2, -1.5, -1, -0.5, 0, 0.5, 1, 1.5, 2],
    "modalityDictionary": { ... },
    "L8dictionary": { ... },
    "S2dictionary": { ... }
  },
  "analysisParameters": {
    "changeThreshold": 0.5,
    "dropThresholdToDenoteChange": 0.59,
    "gainThresholdToDenoteChange": 0.39
  },
  "advancedParameters": {
    "initializationApproach": "unimodal",
    "transitionCreationMethod": "standard"
  }
}
```

### Study Area Geometry

Since JSON can't store `ee.Geometry` objects, use `defaultStudyAreaCoordinates`:

```json
{
  "inputParameters": {
    "defaultStudyAreaCoordinates": [
      [[-123.37, 54.75],
       [-123.37, 54.44],
       [-122.74, 54.44],
       [-122.74, 54.75]]
    ]
  }
}
```

The runner automatically reconstructs the geometry.

## The Environment Switch

The magic happens in your Caller script. This code runs identically in both environments:

```javascript
var finalParams;

if (typeof inputParams !== 'undefined') {
  // LAB MODE: runner11 has injected the JSON
  print(">>> SWITCH: Lab Environment. Using JSON.");
  finalParams = inputParams;
} else {
  // BROWSER MODE: Standard Code Editor
  print(">>> SWITCH: Browser Environment. Loading BULCD-Params.js.");
  finalParams = require('users/alemlakes/BULC-D:BULCD-Params');
}

var inputParameters    = finalParams.inputParameters;
var analysisParameters = finalParams.analysisParameters;
var advanced           = finalParams.advancedParameters;
```

## Module Resolution

The runner maps GEE-style imports to local files:

| GEE Import | Local Path |
|------------|------------|
| `users/alemlakes/BULC-D:BULCD-Params` | `gee_modules/BULC-D/BULCD-Params.js` |
| `users/alemlakes/BULC-Releases:BULC-Module-107` | `gee_modules/BULC-Releases/BULC-Module-107.js` |
| `users/alemlakes/CommonCode:imageCollectionCompression` | `gee_modules/CommonCode/imageCollectionCompression.js` |

## Validation

The runner validates your parameters before execution:

```
✓ Loaded sidecar: high_threshold.json
─────────────────────────────────────────────────────────
► Validating Parameters
─────────────────────────────────────────────────────────
✓ Parameter validation passed
⚠ Validation warnings:
  • dropThreshold < gainThreshold - this may produce unexpected results
```

### Required Keys

**inputParameters:**
- `whichReduction`
- `bandName_reduction`
- `bandNameToFit`
- `binCuts`
- `modalityDictionary`

**analysisParameters:**
- `changeThreshold`
- `dropThresholdToDenoteChange`
- `gainThresholdToDenoteChange`

**advancedParameters:**
- `initializationApproach`
- `transitionCreationMethod`

## Logging

All runs are logged to `logs/` with timestamps:

```
logs/
├── high_threshold_2026-01-21T15-30-45-123Z.log
├── low_threshold_2026-01-21T15-32-12-456Z.log
└── BULCD-Params_2026-01-21T15-28-00-789Z.log
```

## Troubleshooting

### "Module not found" Error

Ensure your local module structure mirrors the GEE repository names:

```bash
# Wrong
gee_modules/my-module/file.js

# Correct (matches users/alemlakes/BULC-D)
gee_modules/BULC-D/file.js
```

### Authentication Errors

```bash
# Check your key file exists
ls -la service-account-key.json

# Or set the path explicitly
export GEE_KEY_PATH=/path/to/key.json
```

### "ee is not defined" in Modules

Ensure your module files use `exports` properly:

```javascript
// Good
exports.myFunction = function() { ... };

// The runner provides `ee` globally - no need to require it
```

## Philosophy

This system follows key principles:

1. **No Parameter Hydration**: We don't merge new values over old defaults. The JSON is complete.
2. **Structural Twins**: The `.js` and `.json` files are mirrors, not partial overlaps.
3. **Environment Detection**: The Caller never "thinks" - it just receives a data package.
4. **Researcher Simplicity**: To run 50 experiments, just modify 50 JSON files.

---

Built for reproducible Earth Engine science. 🌍🛰️
