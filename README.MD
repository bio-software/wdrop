# WDROP - Water Droplet placement around protein surfaces

`wdrop` is a C program that places water molecule oxygens (HOH) around a protein
surface using a PASS-like sphere coating algorithm. It reads a PDB file, deposits
one or more layers of water oxygens on the protein surface, and writes the result
as a new PDB file.

It is used as the water placement step of the MWDROP molecular dynamics
preprocessing pipeline, but has no runtime dependencies of its own beyond the C
standard library and `libm`.

## Compile

```bash
cd src && make          # produces src/wdrop
```

## Install

```bash
cd src && make install  # copies wdrop to ~/bin
```

## Usage

```bash
wdrop --file <input.pdb> [--sigma <radius>] [--weed-dist <dist>] [--layers <n>]
```

| Flag | Meaning | Default |
| --- | --- | --- |
| `-f`, `--file` | Input PDB file (mandatory) | |
| `-s`, `--sigma` | Probe radius | `1.8` |
| `-w`, `--weed-dist` | Minimal O-O distance | `3.5` |
| `-n`, `--layers` | Number of water layers | `1` |
| `-h`, `--help` | Print usage | |

Example:

```bash
./src/wdrop --file src/1PSV.pdb --sigma 1.8 --weed-dist 3.5 --layers 5
```

## Output

- `<input_base>_<N>WAT.pdb` next to the input, where `<N>` is the `--layers` value.
- A log file `<output_pdb>-wdrop-<YYYY-MM-DD HH:MM:SS>` next to it, recording the
  number of added water oxygens and the runtime.

## Testing

```bash
# Determinism: verifies the output is bit-identical across N runs
./testing/algo-deterministic-test/wdrop_deterministic_test.sh [-n <runs>]

# Per-layer runtime benchmark over a directory of PDBs
./testing/layer-runtime-test/run.sh <INPUT_DIR> [OPTIONS]
```

## Acknowledgments

This project reuses code from **MobyWat** http://www.mobywat.com.
