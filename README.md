# Batch Molecular Generation

This project uses RDKit to generate substituted molecular derivatives in batch. It reads a table of molecular cores and a table of substituent fragments, randomly attaches fragments at permitted atom indices, canonicalizes the resulting SMILES, removes duplicates, and exports the products as a CSV file.

The notebook uses project-relative paths only, so the repository can be cloned and run from any location.

## Quick start

```bash
git clone <your-repository-url>
cd molecular-generation
conda create -n molgen -c conda-forge python=3.11 rdkit pandas openpyxl jupyterlab
conda activate molgen
jupyter lab
```

Open `molecular_batch_generation.ipynb` and run its cells in order. `pip install -r requirements.txt` is also supported, but conda-forge is the recommended way to install RDKit reliably.

## Project layout

```
molecular-generation/
├── data/
│   ├── cores.xlsx
│   └── substituents.xlsx
├── output/                         # Created at runtime; CSV results are ignored by Git
├── molecular_batch_generation.ipynb
├── requirements.txt
└── README.md
```

Input Excel files are not included because they can contain proprietary or large datasets. Add only data that is approved for public distribution to GitHub.

## Input format

Both Excel files use the same two-column header:

| smiles | sub_idx |
| --- | --- |
| `c1ccccc1` | `1, 3, (5,7)` |

For `cores.xlsx`, `smiles` is the core SMILES and `sub_idx` lists its substitution-site atom indices. A value such as `(5,7)` represents a symmetric pair of sites.

For `substituents.xlsx`, `smiles` is the substituent-fragment SMILES and `sub_idx` is the zero-based atom index in that fragment that bonds to the core. For example, a fragment whose attachment atom is its first atom uses `0`.

The code always reads the first and second columns by position. Therefore, the recommended headers are `smiles` and `sub_idx`, but a misspelled header will not prevent the notebook from loading the data. Both files must still have at least two columns.

## How generation works

For every selected core, the notebook follows this workflow:

1. Parses the allowed substitution sites from the core table.
2. Builds all non-empty combinations of those sites. To control combinatorial growth, it samples at most ten sites when a core contains more than ten.
3. Randomly chooses one site combination and assigns substituents. All rows in `substituents.xlsx` are automatically eligible; there is no group-ID list to maintain.
4. Treats a parenthesized site pair as symmetric by default, attaching the same substituent to both positions. `NONSYMMETRIC_RATIO` controls how often distinct substituents are used instead.
5. Adds fragments with RDKit, sanitizes each molecule, normalizes the SMILES, selects its largest fragment, neutralizes charges, and removes duplicate canonical SMILES.
6. Stops after reaching the requested count, the attempt limit, or the per-core timeout, then writes `name` and `smiles` columns to `output/generated_molecules.csv`.

Invalid fragment combinations are skipped. If the requested number of unique molecules cannot be generated, the successfully generated subset is still exported and the notebook reports it.

## Configuration

The first code cell contains all user-facing settings. `CORE_INDICES = None` processes every core; use a range or a list such as `range(0, 10)` or `[0, 4, 8]` to select rows. `MOLECULES_PER_CORE` controls the target number of unique products per core.

`RARE_GROUP_IDS` optionally names 1-based rows in `substituents.xlsx` that should be treated as rare. `RARE_GROUP_RATIO` controls their sampling probability. Set `RANDOM_SEED = None` for a different random result on each run; retain an integer seed for reproducible results.

## Output

The notebook replaces `output/generated_molecules.csv` on each run. Copy or rename a result file before the next run if it must be retained. The output directory is excluded from Git by default.
