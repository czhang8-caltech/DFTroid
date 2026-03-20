# Enerdroid

A Swiss-army-knife style Gaussian workflow helper for local post-processing, job checking, and energy table generation.

**Enerdroid** is a Python utility designed for day-to-day Gaussian file handling. It currently supports:

- converting optimized Gaussian `.out` files into single-point `.gjf` inputs
- checking whether Gaussian jobs actually finished successfully
- detecting `.gjf` files without matching `.out`
- building/updating Excel energy summary tables from `xx.out` + `xx_SP.out`

---

## Features

### 1. Convert mode
Convert Gaussian optimization output files into single-point energy input files.

Main behaviors:

- scans all `.out` files in the current directory
- requires:
  - `Optimization completed.`
  - and final Gaussian job status = `SUCCESS`
- extracts:
  - final coordinates
  - charge
  - multiplicity
- writes:
  - `%chk=<output_name>.chk`
- supports output naming rules:
  - `XX.out` → `XX_SP.gjf`
  - `YYZ_BNY.out` → `YYZ_bny_SP.gjf`
- skips existing target `.gjf` files
- remembers the **last route line** and can reuse it next time

---

### 2. Check mode
Check Gaussian output status more reliably than simply searching for error messages.

Sub-functions:

#### 2.1 Check `.out` termination status
Enerdroid classifies Gaussian jobs into three states:

- `SUCCESS`
- `FAILED`
- `UNFINISHED`

This is important because some Gaussian jobs:

- do **not** show explicit error codes
- but still do **not** finish normally

Enerdroid determines job status using the **last termination event** in the output file:

- last event = `Normal termination of Gaussian ...` → `SUCCESS`
- last event = `Error termination ...` → `FAILED`
- no final termination event → `UNFINISHED`

It also tries to extract:

- Gaussian link error codes such as `l301`, `l502`, `l801`
- common Gaussian error messages such as:
  - `QPErr`
  - `End of file in ZSymb`
  - `Problem with the distance matrix`
  - `Convergence failure`
  - etc.

#### 2.2 Check `.gjf` without matching `.out`
This submode scans the current directory and reports `.gjf` files that do not have a corresponding `.out` file with the same stem.

Example:

- `abc.gjf` should match `abc.out`
- `abc_SP.gjf` should match `abc_SP.out`

---

### 3. Statistics mode
Build or update an Excel summary table from successful Gaussian jobs.

Enerdroid scans the current directory for paired files:

- optimization output: `xx.out`
- single-point output: `xx_SP.out`

Only pairs where **both jobs finished successfully** are included.

It extracts:

- `Thermal correction to Enthalpy (TCE)`
- `Thermal correction to Gibbs Free Energy (TCG)`
- `single point energy (SPE)`

And writes an Excel file with columns such as:

- `name`
- `structure`
- `OPT_File`
- `Thermal correction to Enthalpy (TCE)`
- `Thermal correction to Gibbs Free Energy (TCG)`
- `single point energy (SPE)`
- `enthalpies (H)`
- `Gibbs free energies (G)`
- `SP__File`
- `charge`
- `multiplicity`

### Excel behavior
- output file name format:
  - `CurrentFolder_Energy_YYMMDD.xlsx`
- if an older matching Excel already exists:
  - Enerdroid updates from the latest version
  - preserves the `structure` column
  - tries to preserve extra columns, widths, formatting, and manual edits
- `H` and `G` are written as Excel formulas:
  - `H = TCE + SPE`
  - `G = TCG + SPE`

### Sorting behavior
Rows are sorted using a combined rule based on:

- file-name relatedness
- family grouping
- energy proximity

This is intended to keep related structures together, especially cases like:

- `xxy`
- `xxy_V2`
- `xxy_V3`

when both **name similarity** and **energy similarity** suggest they belong together.

---

## Why Enerdroid uses `SUCCESS / FAILED / UNFINISHED`

A Gaussian job should **not** be considered successful merely because it has no explicit error code.

For example, jobs may be:

- killed by walltime limits
- interrupted manually
- stopped by scratch/disk issues
- incomplete for other reasons

In such cases, the output file may:

- contain no `Error termination`
- but also contain no final `Normal termination of Gaussian ...`

Enerdroid therefore uses a stricter standard:

- only files with final `Normal termination of Gaussian ...` are treated as `SUCCESS`

---

## Requirements

- Python 3.10+
- `openpyxl`

Install dependency:

```bash
python -m pip install openpyxl
