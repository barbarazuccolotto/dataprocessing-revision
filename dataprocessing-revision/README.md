# dataprocessing-revision

Pipeline for processing search results exported from bibliographic databases
(Scopus, Web of Science, PubMed, PubMed Central, Dimensions, Google Scholar via
Publish or Perish, Cochrane, Embase, ScienceDirect, IEEE, BVS, CAB and SciELO).

The pipeline runs in two steps, orchestrated by a `Makefile`:

| Step | Script | Input | Output |
|------|--------|-------|--------|
| 1. Format | `scripts/format_input.py` | `data/raw/<type>.<ext>` | `data/01_formatted/input_<type>.xlsx` |
| 2. Remove duplicates | `scripts/remove_duplicates.py` | all files in `data/01_formatted/` | `data/02_deduplicated/summary_screened.xlsx` |

Step 1 converts each export into a standardized format. Step 2 merges the
formatted files, removes duplicate records across databases and records with
incomplete metadata (querying the Crossref REST API when needed), and writes a
single spreadsheet with title, publication year, DOI, document type, language
and author(s).

## Requirements

- Python 3.12 or later
- GNU Make

```sh
make install        # same as: python3 -m pip install -r requirements.txt
```

## Usage

1. Put the exported files in `data/raw/`, **named after the database type**:

   | Database | File name |
   |----------|-----------|
   | Scopus | `scopus.csv` |
   | Web of Science | `wos.csv` |
   | PubMed | `pubmed.csv` |
   | PubMed Central (MEDLINE format) | `pmc.txt` |
   | Dimensions | `dimensions.csv` |
   | Google Scholar (Publish or Perish) | `scholar.csv` |
   | Cochrane | `cochrane.csv` |
   | Embase | `embase.csv` |
   | ScienceDirect | `sciencedirect.ris` |
   | IEEE | `ieee.csv` |
   | BVS | `bvs.csv` |
   | CAB | `cab.csv` |
   | SciELO | `scielo.csv` |
   | Generic text file | `txt.txt` |

   At least two different databases are required for step 2.

2. Run the pipeline:

   ```sh
   make            # runs both steps
   make format     # step 1 only
   make dedup      # step 2 (runs step 1 first if needed)
   make clean      # removes data/01_formatted and data/02_deduplicated
   ```

   Make only re-runs a step when its inputs changed. Each step also writes a
   log file (`run_*.log`) in its output folder.

## Repository structure

```
.
├── Makefile
├── requirements.txt
├── scripts/
│   ├── format_input.py
│   └── remove_duplicates.py
├── data/
│   ├── raw/               # exported files (input)
│   ├── 01_formatted/      # step 1 output (generated)
│   └── 02_deduplicated/   # step 2 output (generated)
└── LICENSES/
```

## Credits and license

The scripts in `scripts/` were developed by Glen Jasper and are distributed
under the MIT License with a required attribution (see `LICENSES/`):

> This software was developed by Glen Jasper (https://github.com/glenjasper),
> originally available at https://github.com/glenjasper/format-input and
> https://github.com/glenjasper/remove-duplicates

Pipeline organization (Makefile and repository structure): Barbara Zuccolotto.
