# Etsy Store Scripts

Python scripts that prepare jig design files for sale as digital downloads on Etsy. They keep the 3D-printing and laser folder trees in sync, add the thank-you PDF to each design, and bundle every design into a consistently structured ZIP file.

## Requirements

- Python 3.10 or newer
- [`python-dotenv`](https://pypi.org/project/python-dotenv/)

```bash
pip install python-dotenv
```

## Configuration

Copy `.env.example` to `.env` in the same directory as the scripts and adjust the paths.

| Key | Used by | Description |
| --- | --- | --- |
| `SYNC_SRC_DIR` | `sync_folders.py` | Source folder whose subfolder names are read. |
| `SYNC_DEST_DIR` | `sync_folders.py` | Destination folder where missing subfolders are created. |
| `ADDIN_SOURCE_DIR` | `sync_digital_addin_files.py` | Folder containing the thank-you PDF(s). |
| `JIGS_3DPRINT_DIR` | `sync_digital_addin_files.py`, `create_etsy_zips.py` | 3D-printing jigs root. |
| `JIGS_LASER_DIR` | `sync_digital_addin_files.py` | Laser jigs root. |
| `ZIP_OUTPUT_DIR` | `create_etsy_zips.py` | Where finished ZIPs are written. Defaults to `JIGS_3DPRINT_DIR`. |
| `ZIP_BACKUP_DIR` | `create_etsy_zips.py` | Where backup ZIPs are stored. Defaults to `JIGS_3DPRINT_DIR/_Backups`. |
| `ZIP_LOG_DIR` | `create_etsy_zips.py` | Where log files are written. Defaults to `JIGS_3DPRINT_DIR`. |
| `ZIP_PREFIX` | `create_etsy_zips.py` | Prefix for every ZIP filename. Defaults to `McQForYouDesign_`. |

## Typical workflow

1. `sync_folders.py`: create any missing design folders in the laser tree.
2. `sync_digital_addin_files.py`: copy the thank-you PDF(s) into each design folder.
3. `create_etsy_zips.py`: build the final ZIPs.

## Scripts

### `sync_folders.py`

Creates any subfolders that exist in `SYNC_SRC_DIR` but not in `SYNC_DEST_DIR`. Existing folders are skipped.

```bash
python sync_folders.py
```

### `sync_digital_addin_files.py`

Copies every PDF from `ADDIN_SOURCE_DIR` into the design subfolders of the chosen jig root(s). Existing copies are overwritten so the latest version is always present. Prompts for any option not passed on the command line.

| Option | Description |
| --- | --- |
| `--jig-type` | `3DPRINT_JIGS`, `LASER_JIGS`, or `BOTH_JIGS`. |
| `--folder NAME` | Only target this subfolder. Defaults to all subfolders. |

```bash
python sync_digital_addin_files.py --jig-type BOTH_JIGS
python sync_digital_addin_files.py --jig-type LASER_JIGS --folder "Bamboo Pen"
```

### `create_etsy_zips.py`

Builds `<ZIP_PREFIX><FolderNameWithoutSpaces>.zip` from each 3D-printing jig folder (non-recursive). Files are sorted into subfolders by type:

```
McQForYouDesign_BambooPen.zip
├── 3MF/    .3mf
├── FBX/    .fbx
├── OBJ/    .obj, .mtl
├── STEP/   .step
├── STL/    .stl
├── *.pdf   (root)
└── *.xcs   (root)
```

Other file types are skipped. Each run reports which expected types (3MF, FBX, OBJ, MTL, STL, PDF, STEP) are missing from a design; XCS is optional and not reported. Output is logged to the console and to a timestamped file in `ZIP_LOG_DIR`, ending with a summary.

| Option | Description |
| --- | --- |
| `--all` | Process every subfolder. |
| `--folder NAME` | Process a single subfolder. |
| `--backup` / `--no-backup` | Create or skip a timestamped backup ZIP of each folder before zipping, without prompting. |

With no options, the script runs interactively.

```bash
python create_etsy_zips.py
python create_etsy_zips.py --all --no-backup
python create_etsy_zips.py --folder "Bamboo Pen" --backup
```
