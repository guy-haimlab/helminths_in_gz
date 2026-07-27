# ================================================================
# Helminths associated with gelatinous zooplankton
# Reproducible R analysis script for publication supporting material
# ================================================================
#
# This script produces:
#   - parasite occurrence donut plot
#   - latitudinal occurrence-frequency plot
#   - latitudinal parasite species-richness plot
#   - parasite-group diversity donut plot
#   - Mediterranean vs Red Sea host-based species accumulation curves
#   - host size vs parasitic load plots and correlation summaries
#
# How to use:
#   1. Place this script in the same folder as the input data files.
#   2. Edit the USER SETTINGS section below if file names differ.
#   3. Run:
#        source("helminth_gelatinous_zooplankton_analysis.R")
#
# Notes:
#   - Missing input files are skipped with a warning, allowing users to run
#     only the analyses for which data are available.
#   - Output files are written to the folder defined by output_dir.
#
# ================================================================


# -----------------------------
# 1. USER SETTINGS
# -----------------------------

input_dir  <- "."
output_dir <- "outputs"

# Input files. Edit these names if your GitHub data files use different names.
files <- list(
  occurrence_donut = "donut.xlsx",
  latlon           = "parlatlon2.xlsx",
  accumulation     = "species accumulation.xlsx",
  host_size        = "host_size_parasitic_load.xlsx"
)

# Excel sheets
sheets <- list(
  occurrence_donut = "Sheet1",
  accumulation_med = "Med",
  accumulation_red = "Red",
  host_size         = 1
)

# Plot settings
bin_size <- 5
dpi_tiff <- 300

# Fixed colour palette for parasite groups
parasite_group_cols <- c(
  "Trematodes"       = "#1B9E77",
  "Cestodes"         = "#D95F02",
  "Nematodes"        = "#7570B3",
  "Acanthocephalans" = "#E7298A"
)

region_cols <- c(
  "Mediterranean" = "#2B6CB0",
  "Red Sea"       = "#C2185B"
)


# -----------------------------
# 2. PACKAGE SETUP
# -----------------------------

required_packages <- c(
  "readxl", "dplyr", "tidyr", "stringr", "ggplot2",
  "scales", "vegan", "purrr", "cowplot", "patchwork"
)

missing_packages <- required_packages[
  !vapply(required_packages, requireNamespace, logical(1), quietly = TRUE)
]

if (length(missing_packages) > 0) {
  stop(
    "Please install the missing packages before running this script:\n",
    "install.packages(c(",
    paste(shQuote(missing_packages), collapse = ", "),
    "))",
    call. = FALSE
  )
}

invisible(lapply(required_packages, library, character.only = TRUE))

if (!dir.exists(output_dir)) {
  dir.create(output_dir, recursive = TRUE)
}


# -----------------------------
# 3. HELPER FUNCTIONS
# -----------------------------

path_in <- function(filename) {
  file.path(input_dir, filename)
}

path_out <- function(filename) {
  file.path(output_dir, filename)
}

file_exists <- function(filename) {
  file.exists(path_in(filename))
}

standardize_colnames <- function(df) {
  names(df) <- names(df) |>
    stringr::str_trim() |>
    stringr::str_replace_all("[^A-Za-z0-9]+", "_") |>
    stringr::str_replace_all("^_|_$", "") |>
    stringr::str_to_lower()
  df
}

latitude_labels <- function(x) {
  ifelse(
    x < 0, paste0(abs(x), "\u00B0S"),
    ifelse(x > 0, paste0(x, "\u00B0N"), "0\u00B0")
  )
}

save_tiff <- function(plot, filename, width = 90, height = 90, dpi = dpi_tiff) {
  ggplot2::ggsave(
    filename = path_out(filename),
    plot = plot,
    width = width,
    height = height,
    units = "mm",
    dpi = dpi,
    compression = "lzw"
  )
}

make_palette <- function(groups, base_palette = parasite_group_cols) {
  groups <- as.character(groups)
  fixed <- base_palette[names(base_palette) %in% groups]
  missing_groups <- setdiff(groups, names(fixed))

  if (length(missing_groups) > 0) {
    extra <- scales::hue_pal()(length(missing_groups))
    names(extra) <- missing_groups
    c(fixed, extra)
  } else {
    fixed
  }
}

safe_cor_test <- function(x, y, method = "spearman") {
  ok <- complete.cases(x, y)
  x <- x[ok]
  y <- y[ok]

  if (length(x) < 3 || stats::sd(x) == 0 || stats::sd(y) == 0) {
    return(tibble::tibble(
      n = length(x),
      estimate = NA_real_,
      p_value = NA_real_
    ))
  }

  test <- suppressWarnings(stats::cor.test(x, y, method = method, exact = FALSE))

  tibble::tibble(
    n = length(x),
    estimate = unname(test$estimate),
    p_value = test$p.value
  )
}


# -----------------------------
# 4. PARASITE OCCURRENCE DONUT
# -----------------------------
# Expected input:
#   File: donut.xlsx
#   Sheet: Sheet1
#   Columns:
#     1. parasite group
#     2. occurrence percentage

plot_occurrence_donut <- function() {
  if (!file_exists(files$occurrence_donut)) {
    warning("Skipping occurrence donut: file not found: ", files$occurrence_donut)
    return(invisible(NULL))
  }

  df <- readxl::read_excel(
    path_in(files$occurrence_donut),
    sheet = sheets$occurrence_donut,
    col_names = TRUE
  )

  df <- df[, 1:2]
  colnames(df) <- c("group", "occurrence_percent")

  donut_df <- df |>
    dplyr::filter(!is.na(group), !is.na(occurrence_percent)) |>
    dplyr::mutate(
      group = stringr::str_squish(as.character(group)),
      occurrence_percent = as.numeric(occurrence_percent)
    ) |>
    dplyr::filter(group != "", !is.na(occurrence_percent)) |>
    dplyr::arrange(dplyr::desc(occurrence_percent))

  cols <- make_palette(unique(donut_df$group))

  p <- ggplot2::ggplot(donut_df, ggplot2::aes(x = 2, y = occurrence_percent, fill = group)) +
    ggplot2::geom_col(width = 1, color = "white", linewidth = 0.5) +
    ggplot2::coord_polar(theta = "y") +
    ggplot2::xlim(0.5, 2.5) +
    ggplot2::scale_fill_manual(values = cols) +
    ggplot2::theme_void(base_size = 12) +
    ggplot2::theme(
      legend.position = "right",
      legend.title = ggplot2::element_blank(),
      legend.text = ggplot2::element_text(size = 11),
      plot.margin = ggplot2::margin(2, 2, 2, 2, "mm")
    ) +
    ggplot2::annotate(
      "text",
      x = 0.5,
      y = 0,
      label = "Parasite\noccurrence (%)",
      size = 4,
      fontface = "bold",
      lineheight = 0.9
    )

  save_tiff(p, "figure_parasite_occurrence_donut.tiff", width = 90, height = 70)
  utils::write.csv(donut_df, path_out("parasite_occurrence_donut_data.csv"), row.names = FALSE)

  invisible(p)
}


# -----------------------------
# 5. LATITUDINAL OCCURRENCE FREQUENCY
# -----------------------------
# Expected input:
#   File: parlatlon2.xlsx
#   Required columns after standardization:
#     lat, lon
#
# Optional columns used elsewhere:
#     species, parasite

prepare_latlon_data <- function() {
  if (!file_exists(files$latlon)) {
    warning("Lat/lon file not found: ", files$latlon)
    return(NULL)
  }

  df <- readxl::read_excel(path_in(files$latlon)) |>
    standardize_colnames()

  # If latitude/longitude are in the first two columns in an older file,
  # use them as lat/lon only when explicit names are missing.
  if (!("lat" %in% names(df)) && ncol(df) >= 1) names(df)[1] <- "lat"
  if (!("lon" %in% names(df)) && ncol(df) >= 2) names(df)[2] <- "lon"

  df |>
    dplyr::mutate(
      lat = as.numeric(lat),
      lon = as.numeric(lon)
    ) |>
    dplyr::filter(
      !is.na(lat), !is.na(lon),
      lat >= -90, lat <= 90,
      lon >= -180, lon <= 180
    )
}

analyse_latitudinal_occurrence <- function() {
  df_clean <- prepare_latlon_data()
  if (is.null(df_clean)) return(invisible(NULL))

  lat_freq <- df_clean |>
    dplyr::mutate(
      lat_mid = floor(lat / bin_size) * bin_size + bin_size / 2
    ) |>
    dplyr::group_by(lat_mid) |>
    dplyr::summarise(
      n_occurrences = dplyr::n(),
      .groups = "drop"
    ) |>
    tidyr::complete(
      lat_mid = seq(-90 + bin_size / 2, 90 - bin_size / 2, by = bin_size),
      fill = list(n_occurrences = 0)
    ) |>
    dplyr::mutate(
      relative_frequency = n_occurrences / sum(n_occurrences)
    )

  p <- ggplot2::ggplot(lat_freq, ggplot2::aes(x = lat_mid, y = relative_frequency)) +
    ggplot2::geom_col(
      width = bin_size * 0.9,
      color = "black",
      linewidth = 0.2,
      fill = "grey80"
    ) +
    ggplot2::geom_smooth(
      method = "loess",
      formula = y ~ x,
      se = FALSE,
      linewidth = 0.9,
      span = 0.35,
      color = "darkgreen"
    ) +
    ggplot2::scale_x_continuous(
      limits = c(-90, 90),
      breaks = seq(-90, 90, by = 30),
      labels = latitude_labels
    ) +
    ggplot2::scale_y_continuous(
      labels = scales::percent_format(accuracy = 1),
      expand = ggplot2::expansion(mult = c(0, 0.08))
    ) +
    ggplot2::labs(
      x = "Latitude",
      y = "Relative frequency of occurrence (%)"
    ) +
    ggplot2::theme_classic(base_size = 12)

  save_tiff(p, "figure_latitudinal_occurrence_frequency_5deg.tiff")
  utils::write.csv(lat_freq, path_out("latitudinal_occurrence_frequency_5deg.csv"), row.names = FALSE)

  invisible(list(data = lat_freq, plot = p))
}


# -----------------------------
# 6. LATITUDINAL PARASITE SPECIES RICHNESS
# -----------------------------
# Expected input:
#   File: parlatlon2.xlsx
#   Required columns after standardization:
#     species, lat, lon

analyse_latitudinal_richness <- function() {
  df_clean <- prepare_latlon_data()
  if (is.null(df_clean)) return(invisible(NULL))

  if (!("species" %in% names(df_clean))) {
    warning("Skipping latitudinal richness: column 'species' was not found in ", files$latlon)
    return(invisible(NULL))
  }

  df_clean <- df_clean |>
    dplyr::mutate(
      species = stringr::str_squish(as.character(species))
    ) |>
    dplyr::filter(!is.na(species), species != "")

  lat_richness <- df_clean |>
    dplyr::mutate(
      lat_mid = floor(lat / bin_size) * bin_size + bin_size / 2
    ) |>
    dplyr::group_by(lat_mid) |>
    dplyr::summarise(
      n_occurrences = dplyr::n(),
      species_richness = dplyr::n_distinct(species),
      .groups = "drop"
    ) |>
    tidyr::complete(
      lat_mid = seq(-90 + bin_size / 2, 90 - bin_size / 2, by = bin_size),
      fill = list(n_occurrences = 0, species_richness = 0)
    ) |>
    dplyr::mutate(
      relative_species_richness = species_richness / sum(species_richness)
    )

  p <- ggplot2::ggplot(lat_richness, ggplot2::aes(x = lat_mid, y = species_richness)) +
    ggplot2::geom_col(
      width = bin_size * 0.9,
      color = "black",
      linewidth = 0.2,
      fill = "grey80"
    ) +
    ggplot2::geom_smooth(
      method = "loess",
      formula = y ~ x,
      se = FALSE,
      linewidth = 0.9,
      span = 0.35,
      color = "purple"
    ) +
    ggplot2::scale_x_continuous(
      limits = c(-90, 90),
      breaks = seq(-90, 90, by = 30),
      labels = latitude_labels
    ) +
    ggplot2::scale_y_continuous(
      expand = ggplot2::expansion(mult = c(0, 0.08))
    ) +
    ggplot2::labs(
      x = "Latitude",
      y = "Helminth species richness"
    ) +
    ggplot2::theme_classic(base_size = 12)

  save_tiff(p, "figure_latitudinal_species_richness_5deg.tiff")
  utils::write.csv(lat_richness, path_out("latitudinal_species_richness_5deg.csv"), row.names = FALSE)

  invisible(list(data = lat_richness, plot = p))
}


# -----------------------------
# 7. PARASITE-GROUP DIVERSITY DONUT
# -----------------------------
# Expected input:
#   File: parlatlon2.xlsx
#   Required columns after standardization:
#     parasite, species

plot_parasite_group_diversity_donut <- function() {
  df <- prepare_latlon_data()
  if (is.null(df)) return(invisible(NULL))

  if (!all(c("parasite", "species") %in% names(df))) {
    warning("Skipping diversity donut: columns 'parasite' and/or 'species' were not found in ", files$latlon)
    return(invisible(NULL))
  }

  diversity_group <- df |>
    dplyr::mutate(
      parasite = stringr::str_squish(as.character(parasite)),
      species = stringr::str_squish(as.character(species))
    ) |>
    dplyr::filter(
      !is.na(parasite), !is.na(species),
      parasite != "", species != ""
    ) |>
    dplyr::group_by(parasite) |>
    dplyr::summarise(
      n_unique_species = dplyr::n_distinct(species),
      .groups = "drop"
    ) |>
    dplyr::mutate(
      diversity_percent = n_unique_species / sum(n_unique_species) * 100,
      label = paste0(
        parasite, "\n",
        n_unique_species, " spp.\n",
        round(diversity_percent, 1), "%"
      )
    ) |>
    dplyr::arrange(dplyr::desc(diversity_percent))

  diversity_group$parasite <- factor(diversity_group$parasite, levels = diversity_group$parasite)
  cols <- make_palette(unique(as.character(diversity_group$parasite)))

  p <- ggplot2::ggplot(
    diversity_group,
    ggplot2::aes(x = 2, y = diversity_percent, fill = parasite)
  ) +
    ggplot2::geom_col(width = 1, color = "white", linewidth = 0.5) +
    ggplot2::coord_polar(theta = "y") +
    ggplot2::xlim(0.5, 2.5) +
    ggplot2::scale_fill_manual(values = cols) +
    ggplot2::theme_void(base_size = 12) +
    ggplot2::theme(
      legend.position = "right",
      legend.title = ggplot2::element_blank(),
      legend.text = ggplot2::element_text(size = 11),
      plot.margin = ggplot2::margin(2, 2, 2, 2, "mm")
    ) +
    ggplot2::annotate(
      "text",
      x = 0.5,
      y = 0,
      label = "Helminth\nspecies diversity",
      size = 3.6,
      fontface = "bold",
      lineheight = 0.9
    )

  save_tiff(p, "figure_parasite_group_diversity_donut.tiff", width = 90, height = 70)
  utils::write.csv(diversity_group, path_out("parasite_group_diversity.csv"), row.names = FALSE)

  invisible(list(data = diversity_group, plot = p))
}


# -----------------------------
# 8. HOST-BASED SPECIES ACCUMULATION CURVES
# -----------------------------
# Expected input:
#   File: species accumulation.xlsx
#   Sheets: Med and Red
#   Required columns:
#     host, parasite
#
# The parasite column can contain multiple parasite names separated by
# semicolons, commas, or line breaks. Hosts with empty parasite cells are
# retained as examined but uninfected hosts.

analyse_species_accumulation <- function() {
  if (!file_exists(files$accumulation)) {
    warning("Skipping species accumulation: file not found: ", files$accumulation)
    return(invisible(NULL))
  }

  med <- readxl::read_excel(path_in(files$accumulation), sheet = sheets$accumulation_med) |>
    standardize_colnames() |>
    dplyr::mutate(region = "Mediterranean")

  red <- readxl::read_excel(path_in(files$accumulation), sheet = sheets$accumulation_red) |>
    standardize_colnames() |>
    dplyr::mutate(region = "Red Sea")

  dat <- dplyr::bind_rows(med, red)

  if (!all(c("host", "parasite") %in% names(dat))) {
    warning("Skipping species accumulation: columns 'host' and/or 'parasite' were not found.")
    return(invisible(NULL))
  }

  dat_clean <- dat |>
    dplyr::mutate(
      region = stringr::str_squish(as.character(region)),
      host = stringr::str_squish(as.character(host)),
      parasite_raw = stringr::str_squish(as.character(parasite))
    ) |>
    dplyr::filter(!is.na(host), host != "") |>
    dplyr::group_by(region) |>
    dplyr::mutate(host_id = paste0("host_", dplyr::row_number())) |>
    dplyr::ungroup()

  all_hosts <- dat_clean |>
    dplyr::select(region, host_id, host)

  dat_long <- dat_clean |>
    dplyr::filter(!is.na(parasite_raw), parasite_raw != "") |>
    tidyr::separate_rows(parasite_raw, sep = ";|\\n|,") |>
    dplyr::mutate(parasite = stringr::str_squish(parasite_raw)) |>
    dplyr::filter(!is.na(parasite), parasite != "") |>
    dplyr::distinct(region, host_id, parasite)

  make_comm_matrix <- function(region_name) {
    hosts_region <- all_hosts |>
      dplyr::filter(region == region_name) |>
      dplyr::select(host_id)

    parasites_region <- dat_long |>
      dplyr::filter(region == region_name)

    if (nrow(parasites_region) == 0) {
      stop("No parasite records found for ", region_name, call. = FALSE)
    }

    comm_df <- parasites_region |>
      dplyr::mutate(presence = 1) |>
      dplyr::select(host_id, parasite, presence) |>
      tidyr::pivot_wider(
        names_from = parasite,
        values_from = presence,
        values_fill = 0
      )

    comm_df <- hosts_region |>
      dplyr::left_join(comm_df, by = "host_id") |>
      dplyr::mutate(dplyr::across(-host_id, ~ tidyr::replace_na(.x, 0)))

    comm <- as.data.frame(comm_df)
    rownames(comm) <- comm$host_id
    comm$host_id <- NULL

    as.matrix(comm)
  }

  set.seed(123)
  regions <- c("Mediterranean", "Red Sea")

  acc_df <- purrr::map_dfr(regions, function(r) {
    comm <- make_comm_matrix(r)

    acc <- vegan::specaccum(
      comm,
      method = "random",
      permutations = 999
    )

    data.frame(
      region = r,
      hosts = acc$sites,
      richness = acc$richness,
      sd = acc$sd
    )
  }) |>
    dplyr::mutate(
      lower = pmax(richness - 1.96 * sd, 0),
      upper = richness + 1.96 * sd
    )

  p <- ggplot2::ggplot(
    acc_df,
    ggplot2::aes(x = hosts, y = richness, colour = region, fill = region)
  ) +
    ggplot2::geom_ribbon(
      ggplot2::aes(ymin = lower, ymax = upper),
      alpha = 0.18,
      colour = NA
    ) +
    ggplot2::geom_line(linewidth = 1.2) +
    ggplot2::geom_point(size = 1.8) +
    ggplot2::scale_colour_manual(values = region_cols) +
    ggplot2::scale_fill_manual(values = region_cols) +
    ggplot2::labs(
      x = "Number of gelatinous hosts examined",
      y = "Accumulated helminth species richness",
      colour = "Region",
      fill = "Region"
    ) +
    ggplot2::theme_classic(base_size = 12) +
    ggplot2::theme(
      legend.position = "top",
      panel.grid.minor = ggplot2::element_blank()
    )

  save_tiff(p, "figure_species_accumulation_Mediterranean_vs_Red_Sea.tiff", width = 100, height = 80)
  utils::write.csv(acc_df, path_out("species_accumulation_Mediterranean_vs_Red_Sea.csv"), row.names = FALSE)

  invisible(list(data = acc_df, plot = p))
}


# -----------------------------
# 9. HOST SIZE VS PARASITIC LOAD
# -----------------------------
# Expected input:
#   File: host_size_parasitic_load.xlsx
#   Required columns after standardization:
#     region
#     host_name
#     host_size_cm
#     parasitic_load
#
# If an R object named host_size_parasitic_load already exists in the
# environment, it will be used only when the Excel file is absent.

read_host_size_data <- function() {
  if (file_exists(files$host_size)) {
    df <- readxl::read_excel(path_in(files$host_size), sheet = sheets$host_size) |>
      standardize_colnames()
  } else if (exists("host_size_parasitic_load", envir = .GlobalEnv)) {
    df <- get("host_size_parasitic_load", envir = .GlobalEnv) |>
      standardize_colnames()
  } else {
    warning(
      "Skipping host-size analysis: neither ",
      files$host_size,
      " nor object 'host_size_parasitic_load' was found."
    )
    return(NULL)
  }

  required <- c("region", "host_name", "host_size_cm", "parasitic_load")
  missing <- setdiff(required, names(df))

  if (length(missing) > 0) {
    warning(
      "Skipping host-size analysis. Missing columns after name standardization: ",
      paste(missing, collapse = ", ")
    )
    return(NULL)
  }

  df |>
    dplyr::transmute(
      region = factor(stringr::str_squish(as.character(region))),
      host = factor(stringr::str_squish(as.character(host_name))),
      size_cm = as.numeric(host_size_cm),
      parasites = as.numeric(parasitic_load),
      log_parasites = log1p(parasites)
    ) |>
    dplyr::filter(
      !is.na(region), !is.na(host),
      !is.na(size_cm), !is.na(parasites)
    )
}

analyse_host_size_parasitic_load <- function() {
  df <- read_host_size_data()
  if (is.null(df)) return(invisible(NULL))

  cor_region <- df |>
    dplyr::group_by(region) |>
    dplyr::group_modify(~ safe_cor_test(.x$size_cm, .x$parasites, method = "spearman")) |>
    dplyr::ungroup() |>
    dplyr::mutate(
      label = paste0(
        "Spearman rho = ", round(estimate, 2),
        "\np = ", signif(p_value, 2)
      )
    )

  cor_host <- df |>
    dplyr::group_by(host) |>
    dplyr::group_modify(~ safe_cor_test(.x$size_cm, .x$parasites, method = "spearman")) |>
    dplyr::ungroup() |>
    dplyr::mutate(
      label = paste0(
        "rho = ", round(estimate, 2),
        "\np = ", signif(p_value, 2)
      )
    )

  utils::write.csv(cor_region, path_out("host_size_correlations_by_region.csv"), row.names = FALSE)
  utils::write.csv(cor_host, path_out("host_size_correlations_by_host.csv"), row.names = FALSE)

  # Region-level plot: host-specific lines plus an overall regional trend.
  p_region <- ggplot2::ggplot(df, ggplot2::aes(x = size_cm, y = log_parasites, colour = host)) +
    ggplot2::geom_point(alpha = 0.8, size = 2, na.rm = TRUE) +
    ggplot2::geom_smooth(method = "lm", se = FALSE, linewidth = 0.5, na.rm = TRUE) +
    ggplot2::geom_smooth(
      ggplot2::aes(group = 1),
      method = "lm",
      se = FALSE,
      colour = "black",
      linewidth = 0.8,
      na.rm = TRUE
    ) +
    ggplot2::geom_text(
      data = cor_region,
      ggplot2::aes(x = -Inf, y = Inf, label = label),
      inherit.aes = FALSE,
      hjust = -0.05,
      vjust = 1.1,
      size = 3.5,
      colour = "black"
    ) +
    ggplot2::facet_wrap(~ region, nrow = 1) +
    ggplot2::labs(
      x = "Host size (cm)",
      y = "log(Parasitic load + 1)",
      colour = "Host"
    ) +
    ggplot2::theme_classic(base_size = 12) +
    ggplot2::theme(
      strip.text = ggplot2::element_text(size = 13, face = "bold"),
      legend.position = "right"
    )

  save_tiff(p_region, "figure_host_size_parasitic_load_by_region.tiff", width = 150, height = 80)

  # Host-level plot. Keep hosts with enough data and at least one parasite.
  valid_hosts <- df |>
    dplyr::group_by(host) |>
    dplyr::summarise(
      n = dplyr::n(),
      max_parasites = max(parasites, na.rm = TRUE),
      .groups = "drop"
    ) |>
    dplyr::filter(n > 5, max_parasites > 0)

  df_host <- df |>
    dplyr::semi_join(valid_hosts, by = "host")

  if (nrow(df_host) > 0) {
    cor_host_plot <- cor_host |>
      dplyr::semi_join(valid_hosts, by = "host")

    p_host <- ggplot2::ggplot(
      df_host,
      ggplot2::aes(x = size_cm, y = log_parasites, colour = region)
    ) +
      ggplot2::geom_point(size = 1.8, alpha = 0.8, na.rm = TRUE) +
      ggplot2::geom_smooth(method = "lm", se = FALSE, colour = "black", linewidth = 0.6, na.rm = TRUE) +
      ggplot2::geom_text(
        data = cor_host_plot,
        ggplot2::aes(x = -Inf, y = Inf, label = label),
        inherit.aes = FALSE,
        hjust = -0.05,
        vjust = 1.1,
        size = 3,
        colour = "black"
      ) +
      ggplot2::facet_wrap(~ host, scales = "free_x") +
      ggplot2::scale_colour_manual(values = region_cols) +
      ggplot2::labs(
        x = "Host size (cm)",
        y = "log(Parasitic load + 1)",
        colour = "Region"
      ) +
      ggplot2::theme_classic(base_size = 11) +
      ggplot2::theme(
        strip.text = ggplot2::element_text(size = 10, face = "bold"),
        legend.position = "bottom"
      )

    save_tiff(p_host, "figure_host_size_parasitic_load_by_host.tiff", width = 170, height = 120)
  }

  # Optional count model for parasitic load.
  # This is run only if the MASS package is installed and the dataset has enough records.
  if (requireNamespace("MASS", quietly = TRUE) && nrow(df) >= 10) {
    m_nb <- tryCatch(
      MASS::glm.nb(parasites ~ size_cm + host + region, data = df),
      error = function(e) e
    )

    capture.output(
      {
        cat("Negative binomial model: parasites ~ size_cm + host + region\n\n")
        if (inherits(m_nb, "error")) {
          cat("Model failed:\n")
          print(m_nb$message)
        } else {
          print(summary(m_nb))
        }
      },
      file = path_out("negative_binomial_model_host_size_parasitic_load.txt")
    )
  }

  invisible(list(
    data = df,
    cor_region = cor_region,
    cor_host = cor_host,
    plot_region = p_region
  ))
}


# -----------------------------
# 10. RUN ALL ANALYSES
# -----------------------------

run_all_analyses <- function() {
  message("Running helminth / gelatinous zooplankton analyses...")

  results <- list(
    occurrence_donut       = plot_occurrence_donut(),
    latitudinal_occurrence = analyse_latitudinal_occurrence(),
    latitudinal_richness   = analyse_latitudinal_richness(),
    diversity_donut        = plot_parasite_group_diversity_donut(),
    species_accumulation   = analyse_species_accumulation(),
    host_size              = analyse_host_size_parasitic_load()
  )

  message("Done. Output files were written to: ", normalizePath(output_dir, mustWork = FALSE))

  invisible(results)
}

results <- run_all_analyses()
