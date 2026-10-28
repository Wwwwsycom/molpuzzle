# Input data

Place `cores.xlsx` and `substituents.xlsx` in this directory. The files are intentionally excluded from the repository unless they are approved for publication.

Both files use this header format:

| smiles | sub_idx |
| --- | --- |

`cores.xlsx` stores one core SMILES and its substitution-site specification per row. `sub_idx` may contain individual atom indices and symmetric pairs, for example `1, 3, (5,7)`.

`substituents.xlsx` stores one substituent-fragment SMILES and the zero-based index of its attachment atom per row. Its row number (starting at 1) is the substituent ID used only when configuring `RARE_GROUP_IDS`.

The notebook reads the first two columns by position, so it can still load files with incorrect headers. The recommended headers above make the files understandable to people and other tools.
