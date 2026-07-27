[README_helminth_analysis_supporting_material.md](https://github.com/user-attachments/files/30407531/README_helminth_analysis_supporting_material.md)
# Helminths associated with gelatinous zooplankton: R analysis supporting material

This folder contains a cleaned R script for the analyses used in the manuscript on helminths associated with gelatinous zooplankton.

## Main script

`helminth_gelatinous_zooplankton_analysis.R`

## Expected input files

Place the input files in the same folder as the R script, or edit `input_dir` and the filenames in the `USER SETTINGS` section.

| File | Purpose | Expected columns/sheets |
|---|---|---|
| `donut.xlsx` | Parasite occurrence donut plot | Sheet `Sheet1`; first column = parasite group, second column = occurrence percentage |
| `parlatlon2.xlsx` | Latitudinal occurrence/richness and parasite-group diversity | Columns including `species`, `parasite`, `lat`, `lon` |
| `species accumulation.xlsx` | Species accumulation curves | Sheets `Med` and `Red`; columns `host`, `parasite` |
| `host_size_parasitic_load.xlsx` | Host size vs parasitic load | Columns `region`, `host name`, `host size (cm)`, `parasitic load` |

## How to run

From R or RStudio:

```r
source("helminth_gelatinous_zooplankton_analysis.R")
```

The script writes output figures and summary tables to the `outputs/` folder.

## Main outputs

- `figure_parasite_occurrence_donut.tiff`
- `figure_latitudinal_occurrence_frequency_5deg.tiff`
- `figure_latitudinal_species_richness_5deg.tiff`
- `figure_parasite_group_diversity_donut.tiff`
- `figure_species_accumulation_Mediterranean_vs_Red_Sea.tiff`
- `figure_host_size_parasitic_load_by_region.tiff`
- `figure_host_size_parasitic_load_by_host.tiff`
- CSV summary tables for latitudinal bins, parasite diversity, accumulation curves, and correlations

## Notes

- The script does not automatically install packages. Missing packages are reported at the beginning with an installation command.
- Missing input files are skipped with a warning so that partial analyses can still be run.
- The script is intended as supporting material and may require minor filename edits if the deposited data files are renamed.
