## The Commuting–Environment Nexus
#### Traffic Congestion, Transit Use, and Air Quality in 10 Major U.S. Cities (2018–2024)

**Author:** Salman Rashid
**Course:** Econ 396 — Final Project
**Output:** `Final Project.Rmd` → PDF report

---

### 1. Overview

This project builds a multi-source, city-by-year (panel) dataset for 10 major U.S.
metropolitan areas and tests whether traffic congestion and public transit use are
associated with ambient PM2.5 air pollution, controlling for income, population
density, and summer temperature.

**Hypothesis:** Cities with higher traffic congestion and lower public transit
usage tend to experience worse air quality (higher PM2.5), even after controlling
for income, population density, and summer temperature.

The analysis combines:

| Source | Variable(s) | Method |
|---|---|---|
| INRIX Global Traffic Scorecard | Annual hours lost to congestion (2018–2024) | Manually transcribed from published scorecards |
| U.S. Census Bureau (ACS 5-year) | % transit commuters, median income, population density | Census API (JSON) |
| EPA Air Quality System (AQS) | Annual mean PM2.5 (µg/m³) | AQS `annualData/byCBSA` API |
| NOAA Climate Data Online | Summer (Jun–Aug) average temperature | CDO GHCND daily data |

All sources are merged into a single tidy panel (`master_data`, one row per
city-year) and analyzed via a scatterplot, bar chart, correlation matrix, and a
pooled OLS regression. All interpretive text in the report (correlation
direction, regression coefficient signs/significance, etc.) is computed
dynamically from the results, so the narrative always matches the output.

---

### 2. Repository Contents

```
.
├── Econ-396-Final-Project.Rproj   # RStudio project file
├── Final Project.Rmd              # Main analysis (knit to PDF)
├── .Renviron                      # API credentials (NOT committed — see §3.3)
├── .gitignore                     # Excludes secrets, R artifacts, rendered output
└── README.md                      # This file
```

Knitting `Final_Project.Rmd` produces a self-contained PDF report containing the
written description, methodology, data tables, visualizations, regression output,
and reflection.

---

### 3. Setup

#### 3.1 Software Requirements
- R ≥ 4.2
- RStudio (recommended, via the `.Rproj` file)
- A LaTeX distribution for PDF rendering (e.g. TinyTeX:
  `tinytex::install_tinytex()`)

#### 3.2 R Packages

```r
install.packages(c(
  "tidyverse", "knitr", "broom",
  "corrplot", "httr", "jsonlite", "styler"
))
```

#### 3.3 API Credentials

This project pulls live data from three external APIs at knit time. Credentials
are read from environment variables (never hardcoded) via `Sys.getenv()`. You'll
need your own free API keys:

| Service | Used for | Get a key |
|---|---|---|
| U.S. Census Bureau | ACS demographic data | https://api.census.gov/data/key_signup.html |
| EPA AQS API | PM2.5 air quality data | https://aqs.epa.gov/data/api/signup |
| NOAA CDO API | Summer temperature data | https://www.ncdc.noaa.gov/cdo-web/token |

**Create a `.Renviron` file** in your home directory or project root:

```r
file.edit("~/.Renviron")
```

Add the following (no quotes, no spaces around `=`):

```
CENSUS_API_KEY=your_census_key
EPA_API_EMAIL=your_email@example.com
EPA_API_KEY=your_epa_key
NOAA_API_TOKEN=your_noaa_token
```

Save, then restart R (**Session → Restart R**) or run
`readRenviron("~/.Renviron")` to load the variables. Verify with:

```r
Sys.getenv("CENSUS_API_KEY")
```

`.Renviron` is excluded from version control via `.gitignore` and should never be
committed or shared.

---

### 4. How to Run

1. Open `Econ-396-Final-Project.Rproj` in RStudio (sets the working directory
   automatically).
2. Confirm your `.Renviron` is set up and loaded (§3.3).
3. Install required packages (§3.2) if needed.
4. Knit `Final_Project.Rmd` (click **Knit**, or run
   `rmarkdown::render("Final_Project.Rmd")`).
5. The rendered PDF appears in the project directory.

**Expected runtime:** several minutes, due to rate-limited API loops over 10
cities × 7 years (2018–2024) for the EPA and NOAA endpoints (`Sys.sleep()` calls
are built in to respect each API's rate limits).

---

### 5. Code Style

Code is formatted with [`styler`](https://styler.r-lib.org/) following the
tidyverse style guide. To re-format after edits:

```r
library(styler)
style_file("Final Project.Rmd")
```

---

### 6. Methodology Notes

- **INRIX congestion data** is manually transcribed from INRIX's published annual
  scorecards using a consistent "hours lost vs. free-flow" methodology
  (2018-onward only; pre-2018 figures use a different metric and are excluded).
- **Unbalanced panel:** Several city-years are missing from the INRIX series
  (notably Dallas, Houston, Los Angeles, and Seattle for 2018–2022). After merging
  all four sources and dropping incomplete rows, the final panel is unbalanced —
  this is reported transparently rather than imputed.
- **Tidy format:** `master_data` follows tidy data principles (Wickham, 2014) —
  one row per city-year, one column per variable, one value per cell.
- **Model:** Pooled OLS regression of annual mean PM2.5 on congestion hours,
  transit use, median income, population density, and summer temperature. This
  is an associational, not causal, model.

---

### 7. Results Summary

- The bivariate relationship between average congestion and average PM2.5 is
  **weak and negative** (r ≈ -0.06) — not the positive relationship the
  hypothesis predicted.
- In the pooled OLS regression, the congestion-hours coefficient is **negative
  and not statistically significant**, so the data show no detectable
  relationship between congestion and PM2.5 once other factors are controlled
  for.
- The transit-use coefficient points in the hypothesized (negative) direction
  but is also not statistically significant in this sample.
- **Overall:** the project does not find strong support for the original
  hypothesis. It is best read as an exploratory first pass; a panel model with
  city fixed effects and a larger, more balanced sample would be the natural next
  step.

---

### 8. Known Limitations

- Results depend on live API responses at knit time. EPA AQS and NOAA CDO
  occasionally return empty results for certain station/CBSA-year combinations;
  these are logged and dropped during the final merge, so sample size can vary
  slightly between runs.
- Small sample (n = 10 cities, unbalanced 2018–2024 panel) — results are
  exploratory, not confirmatory.
- Pooled OLS does not account for city-level unobserved heterogeneity; a
  fixed-effects or first-differenced panel model is recommended as a next step.

---

### 9. References

- EPA (2024). *Air Quality System (AQS) API.* https://aqs.epa.gov/aqsweb/documents/data_api.html
- INRIX (2018–2024). *Global Traffic Scorecard.* https://inrix.com/scorecard/
- NOAA (2024). *Climate Data Online (CDO) Web Services.* https://www.ncei.noaa.gov/cdo-web/
- U.S. Census Bureau (2018–2024). *American Community Survey 5-year estimates.* https://www.census.gov/data/developers/data-sets/acs-5year.html
- Wickham, H. (2014). Tidy data. *Journal of Statistical Software, 59*(10), 1–23.

---

## 10. License

For academic use as part of Econ 396. Contact the author for reuse permissions.
