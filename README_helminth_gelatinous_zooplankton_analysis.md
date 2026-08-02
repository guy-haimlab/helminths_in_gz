# Helminths associated with gelatinous zooplankton: R analysis workflow

This repository contains an R script for reproducing the main analyses and figures for the study of helminths associated with gelatinous zooplankton. The script generates parasite-composition plots, latitudinal occurrence and richness plots, Red Sea vs Mediterranean species-accumulation curves, host-size vs parasite-load analyses, and permutational sensitivity analyses of the latitudinal pattern.

## Files

Place the R script and all input files in the same working folder, unless you edit `input_dir` in the script.

Default input files expected by the script:

| Input file | Purpose | Required columns / structure |
|---|---|---|
| `donut.xlsx` | Parasite occurrence donut plot | First column: parasite group; second column: occurrence percentage |
| `parlatlon2.xlsx` | Latitudinal occurrence, species richness, parasite-group diversity, and sensitivity analyses | `lat`, `lon`; additionally `species` for richness analyses and `parasite` for parasite-group diversity |
| `species accumulation.xlsx` | Host-based species accumulation curves | Sheets named `Med` and `Red`; columns `host` and `parasite` |
| `host size parasitic load.xlsx` | Host size vs parasitic load analyses | `region`, `host_name`, `host_size_cm`, `parasitic_load` |

Output files are written to the folder specified by `output_dir`, which is set by default to:

```r
output_dir <- "outputs"
```

If the folder does not exist, the script creates it automatically.

## Required R packages

The script requires the following R packages:

```r
install.packages(c(
  "readxl", "dplyr", "tidyr", "stringr", "ggplot2",
  "scales", "vegan", "purrr", "cowplot", "patchwork"
))
```

The optional negative-binomial model for host size vs parasitic load uses the `MASS` package when it is available.

## How to run

1. Open R or RStudio.
2. Set the working directory to the folder containing the script and input files.
3. Run:

```r
source("helminth_gelatinous_zooplankton_analysis.R")
```

If your script has a different filename, replace the filename in the `source()` command accordingly.

The script automatically runs all analyses through:

```r
results <- run_all_analyses()
```

The returned object `results` stores the main data tables and plots generated during the run.

## User settings

At the top of the script, edit the following settings if file names, folders, or analysis options differ:

```r
input_dir  <- "."
output_dir <- "outputs"

files <- list(
  occurrence_donut = "donut.xlsx",
  latlon           = "parlatlon2.xlsx",
  accumulation     = "species accumulation.xlsx",
  host_size        = "host size parasitic load.xlsx"
)

bin_size <- 5
dpi_tiff <- 300
```

The default latitudinal bin size is 5°.

## Permutational sensitivity analysis

The script includes a permutational sensitivity analysis to test whether the inferred temperate / inverse latitudinal diversity pattern is robust to:

1. alternative latitudinal bin widths,
2. random shifts in bin boundaries,
3. small uncertainty in locality latitude,
4. record-level resampling.

These settings are controlled near the top of the script:

```r
run_sensitivity_analysis <- TRUE
sensitivity_permutations <- 9999
sensitivity_bin_widths <- c(5, 10, 15)
latitude_jitter_deg <- 2.5
```

For faster testing, reduce the number of permutations, for example:

```r
sensitivity_permutations <- 999
```

For the final manuscript analysis, use a larger number of permutations, such as 9,999, and report the final values from `sensitivity_latitudinal_pattern_summary.csv`.

To skip the longer permutation analyses, set:

```r
run_sensitivity_analysis <- FALSE
```

## Main analyses and outputs

### 1. Parasite occurrence donut plot

Function:

```r
plot_occurrence_donut()
```

Outputs:

- `figure_parasite_occurrence_donut.tiff`
- `parasite_occurrence_donut_data.csv`

### 2. Latitudinal occurrence frequency

Function:

```r
analyse_latitudinal_occurrence()
```

This analysis calculates the number and relative frequency of helminth occurrence records in latitudinal bins.

Outputs:

- `figure_latitudinal_occurrence_frequency_5deg.tiff`
- `latitudinal_occurrence_frequency_5deg.csv`

### 3. Latitudinal parasite species richness

Function:

```r
analyse_latitudinal_richness()
```

This analysis calculates helminth species richness per latitudinal bin using unique species names.

Outputs:

- `figure_latitudinal_species_richness_5deg.tiff`
- `latitudinal_species_richness_5deg.csv`

### 4. Permutational sensitivity analysis of latitudinal patterns

Function:

```r
analyse_latitudinal_sensitivity()
```

This analysis resamples records, jitters latitude values, shifts bin boundaries, and tests alternative bin widths. It summarizes whether occurrence and richness peaks remain in temperate latitudes and whether temperate latitudes exceed tropical latitudes.

Outputs:

- `sensitivity_latitudinal_pattern_replicates.csv`
- `sensitivity_latitudinal_pattern_summary.csv`
- `figure_sensitivity_temperate_tropical_richness_ratio.tiff`

### 5. Null model for latitudinal species richness

Function:

```r
analyse_latitudinal_richness_null()
```

This null model tests whether observed species richness per latitude bin can be explained by uneven numbers of occurrence records among bins. It keeps the observed number of records per bin fixed while randomly permuting parasite species identities among records.

Outputs:

- `sensitivity_species_richness_null_by_bin.csv`
- `figure_latitudinal_richness_null_model.tiff`

### 6. Parasite-group diversity donut plot

Function:

```r
plot_parasite_group_diversity_donut()
```

This analysis calculates the number of unique parasite species in each parasite group.

Outputs:

- `figure_parasite_group_diversity_donut.tiff`
- `parasite_group_diversity.csv`

### 7. Mediterranean vs Red Sea host-based species accumulation curves

Function:

```r
analyse_species_accumulation()
```

This analysis uses host-level parasite presence/absence matrices and `vegan::specaccum()` to compare accumulated helminth species richness as a function of the number of gelatinous hosts examined.

Outputs:

- `figure_species_accumulation_Mediterranean_vs_Red_Sea.tiff`
- `species_accumulation_Mediterranean_vs_Red_Sea.csv`

### 8. Host size vs parasitic load

Function:

```r
analyse_host_size_parasitic_load()
```

This analysis evaluates relationships between host size and parasite load using Spearman correlations by region and by host. When the `MASS` package is available and the dataset is sufficiently large, the script also attempts a negative-binomial model.

Outputs:

- `host_size_correlations_by_region.csv`
- `host_size_correlations_by_host.csv`
- `figure_host_size_parasitic_load_by_region.tiff`
- `figure_host_size_parasitic_load_by_host.tiff`
- `negative_binomial_model_host_size_parasitic_load.txt` when the model runs successfully

## Notes on missing files

The script is designed to skip analyses when required input files are missing. Missing files produce warnings rather than stopping the entire workflow. This allows users to run only the analyses for which data are currently available.

## Troubleshooting

### The script is slow

The sensitivity analyses can take time because they may run 9,999 permutations for both the resampling sensitivity analysis and the richness null model. For testing, reduce:

```r
sensitivity_permutations <- 999
```

or skip the sensitivity analyses:

```r
run_sensitivity_analysis <- FALSE
```

### A plot or output file was not generated

Check whether the required input file exists and whether the required columns are present after column-name standardization. The script standardizes column names by trimming spaces, replacing non-alphanumeric characters with underscores, and converting names to lowercase.

### Latitude or longitude values are removed

The script keeps only records with valid coordinates:

- latitude between -90 and 90,
- longitude between -180 and 180.

Records outside these ranges or with missing coordinates are filtered out.

## Recommended citation / reuse note

When sharing the script and outputs, cite the associated manuscript and include the version of the input data files used to generate the results.
