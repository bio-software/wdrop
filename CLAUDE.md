# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

WDROP is a C program (`wdrop`) that places water molecule oxygens (HOH) around a protein surface using a PASS-like sphere coating algorithm. It is consumed as the water placement step by the separate MWDROP pipeline (GROMACS/Tinker MM+MD, MobyWat), which lives in its own repository.

## Build

```bash
cd src && make          # produces src/wdrop
make install            # copies wdrop to ~/bin
```

CI runs `make` from `src/` on every push/PR (`.github/workflows/c-cpp.yml`).

## Running wdrop

```bash
wdrop --file <input.pdb> --sigma <float> --weed-dist <float> --layers <int>
```

Output is written as `<input_base>_<N>WAT.pdb` in the same directory as the input. A log entry is also written alongside it.

## Testing

```bash
# Determinism test (verifies the C program output is bit-identical across N runs):
./testing/algo-deterministic-test/wdrop_deterministic_test.sh [-n <runs>]

# Per-layer runtime benchmark (chains --layers 1 runs, records wdrop's logged runtime to CSV):
./testing/layer-runtime-test/run.sh <INPUT_DIR> [OPTIONS]
```

`layer-runtime-test/run.sh --preprocess` expects `pipeline/pdb-preprocessor.py`, which belongs to the MWDROP repository and is not present here; without `--preprocess` the script only needs `src/wdrop`.

## Code architecture (`src/`)

- **`macros.h`** — all compile-time constants and helper print macros. PDB field widths, default tolerances, program name strings.
- **`global.h`** — all PDB-related structs: `ap` (atom), `tp` (topology), `pd` (pairwise distance), `hs`/`hd` (hydrogen-bond network edge/node), `re`/`pl`/`pf` (pool/occupancy lists), `wr`/`mr` (water matching). Struct fields reflect the PDB column layout.
- **`input.c`/`input.h`** — `read_in_pdb()` parses PDB files into `ap[]` arrays; `parse_program_args()` handles CLI flags (`--file`, `--sigma`, `--weed-dist`, `--layers`) into `WdropProgramOptions`.
- **`wdrop.c`/`wdrop.h`** — core algorithm. `three_point_sphere_geometry()` computes candidate water positions from every triple of nearby atom spheres; `pass_like_coating()` iterates over atom triples, filters clashes, and appends accepted HOH atoms. Uses a linked-list spatial grid for neighbor lookup. `sigma_from_atom()` maps element symbols to van-der-Waals radii.
- **`print.c`/`print.h`** — PDB file output and per-run log entry.
- **`main.c`** — entry point: parse args → read PDB → `pass_like_coating()` → write output PDB + log.

Global state lives in `main.c` as `g_`-prefixed variables (e.g. `g_pdb_ref`, `g_ref_name`). The structs in `global.h` carry extra fields inherited from MobyWat that are not used by `wdrop` itself.

### Shell coding conventions (test scripts)

- Variable names: `SCREAMING_SNAKE_CASE` throughout.
- Local variables declared with `local`.
- Scripts use `set -euo pipefail`.
