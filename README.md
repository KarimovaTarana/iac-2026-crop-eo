# Satellite-Informed Climate Analysis of Asian Crop Yield Vulnerability

Analysis code and derived data for the IAC 2026 paper *"Satellite-Informed Climate Analysis of Asian Crop Yield Vulnerability: Space-Enabled Pathways for Agricultural Resilience"* (IAC-26-[PAPER CODE]), presented at the 77th International Astronautical Congress, Antalya, Türkiye, 5–9 October 2026.

**Author:** Tarana Karimova
**Contact:** kerimova.terane2004@gmail.com
**Archived version:** [Zenodo DOI]

---

## What this does

Links FAOSTAT crop production statistics for rice, wheat and maize to four independent satellite and climate data streams across 16 Asian countries, producing a panel of 6,003 country–crop–year observations. Estimates temperature sensitivity of detrended yields, tests how that sensitivity varies across sub-regions, and assesses the marginal contribution of satellite-derived predictors over station-based climate records.

All input data are publicly available at no cost. The analysis requires no API keys or paid subscriptions.

---

## Quick start

```bash
git clone https://github.com/[USERNAME]/iac-2026-crop-eo.git
cd iac-2026-crop-eo
pip install -r requirements.txt
jupyter lab notebooks/analysis.ipynb
```

Then **Kernel → Restart Kernel and Run All Cells**.

The notebook runs end to end from the included `data/processed/panel.csv` and `data/processed/vhp_long.csv`. Re-downloading raw satellite data is **not** required to reproduce the published tables and figures.

---

## Repository structure

```
├── notebooks/
│   └── analysis.ipynb           Complete pipeline: load → clean → merge → analyse
├── data/
│   ├── raw/
│   │   ├── faostat/             FAOSTAT bulk downloads (QCL, ET, RL)
│   │   ├── gistemp/             GISTEMP v4 zonal annual means
│   │   ├── appeears/            MODIS LST/NDVI and SMAP point extractions
│   │   ├── vhp/                 55 NOAA VHP administrative-unit time series
│   │   └── chirps/              CHIRPS v3.0 GeoTIFFs (not tracked — see below)
│   └── processed/
│       ├── panel.csv            ★ The analysis panel (6,003 × 24)
│       ├── vhp_long.csv         Parsed VHP weekly series (124,905 rows)
│       └── chirps_points.csv    CHIRPS sampled at the 20 coordinates
├── outputs/
│   ├── figures/                 Figures 1–6 as published
│   └── tables/                  Results tables as CSV
├── requirements.txt
└── README.md
```

**`data/raw/chirps/` is not tracked.** The CHIRPS GeoTIFFs total roughly 3 GB and are freely redistributable from the source. `data/processed/chirps_points.csv` contains the sampled values actually used, so the rasters are only needed if you wish to re-sample at different coordinates.

---

## Data sources

| Dataset | Product | Coverage | Source |
|---|---|---|---|
| Crop yield | FAOSTAT QCL, Asia | 1961–2024 | https://www.fao.org/faostat/ |
| Temperature | FAOSTAT ET (GISTEMP-derived) | 1961–2024 | https://www.fao.org/faostat/ |
| Land use | FAOSTAT RL | 1961–2024 | https://www.fao.org/faostat/ |
| Warming context | GISTEMP v4 zonal annual means | 1880–2025 | https://data.giss.nasa.gov/gistemp/ |
| Vegetation condition | NOAA STAR VHP (VCI, TCI, VHI) | 1982–2025 | https://www.star.nesdis.noaa.gov/smcd/emb/vci/VH/ |
| Land surface temperature | MOD11A2 v061 (Terra) | 2000–2025 | NASA LP DAAC via AppEEARS |
| Vegetation index | MOD13A2 v061 (Terra) | 2000–2025 | NASA LP DAAC via AppEEARS |
| Precipitation | CHIRPS v3.0 monthly | 1981–2025 | https://data.chc.ucsb.edu/products/CHIRPS/v3.0/ |
| Soil moisture | SMAP SPL3SMP_E V6 | 2015–2025 | NASA NSIDC DAAC via AppEEARS |

Full citations with DOIs appear in the paper's reference list.

### Reproducing the raw data acquisition

**FAOSTAT** — download the Asia regional bulk files from the QCL, ET and RL domains; place the extracted `*_NOFLAG.csv` files under `data/raw/faostat/`.

**GISTEMP** — download `ZonAnn.Ts+dSST.csv` to `data/raw/gistemp/`.

**NOAA VHP** — the 55 administrative-unit series are included in `data/raw/vhp/`. To regenerate, use the browse-by-country interface with Year1 = 1981, Year2 = 2026 and save the "Area-Averaged" ASCII output. Files are named `{ISO3}_{Unit}.csv`; the parser derives the country code from the text before the first underscore.

**AppEEARS** — submit a point-sample request using the 20 coordinates in Appendix B of the paper, selecting layers `MOD11A2.061 LST_Day_1km`, `MOD13A2.061 _1_km_16_days_NDVI` and `SPL3SMP_E Soil_Moisture_Retrieval_Data_AM_soil_moisture`, dates 2000-01-01 to 2025-12-31, CSV output.

**CHIRPS** — monthly global GeoTIFFs for February–April, 1981–2025, from the v3.0 directory. Sampled at the same 20 coordinates using `rasterio`.

---

## Notebook sections and paper outputs

| Notebook section | Produces |
|---|---|
| 0. Setup | Paths, plotting defaults |
| 1. Load raw data | All source frames; VHP and CHIRPS cached to `processed/` |
| 2. Inspect | Data quality checks, element and item strings |
| 3. Build panel | `panel.csv` — melt, filter, detrend, seasonal aggregation, merge |
| 4.1 | **Table 2** — yield volatility by sub-region and crop |
| 4.2 | **Table 3** — pooled temperature sensitivity |
| 4.3 | **§5.5** — precipitation control |
| 4.4 | **Table 5** — seasonal vs annual temperature windows |
| 4.5 | **Table 4** — sub-regional interaction model |
| 4.6 | **Table 6** — EO marginal contribution, three specifications |
| 4.7 | **Table 7** — irrigation interaction |
| 5. Figures | **Figures 1–6** |

**Table 1** (VHI diagnostic) comes from the index-selection cells in section 4.

---

## Methodological notes

**Yield detrending.** Yields are detrended within each country–crop series using a second-order polynomial in *log* yield. The log form avoids the numerical instability of percentage deviation where the fitted trend approaches zero; an earlier percentage-based specification produced implausible volatility for Central/West Asian maize.

**Vegetation indicator.** The analysis uses VCI rather than the composite VHI. VHI's thermal component (TCI) is structurally depressed in humid tropical Asia, where growing-season temperatures sit near their climatological maximum every year, producing spurious drought signals for Viet Nam, Cambodia and Bangladesh. See §4.4 and §6.2 of the paper.

**Sample identity in nested comparisons.** Model comparisons drop observations missing any variable in *either* model before estimation, so R² and F-tests compare like with like.

**Country aggregation.** FAOSTAT reports `China` as an aggregate including Taiwan Province, Hong Kong SAR and Macao SAR. The aggregate is dropped and `China, mainland` retained.

---

## Known limitations

Documented fully in §7 of the paper. In brief: yield data are national, so sub-national heterogeneity is averaged out; MODIS and CHIRPS are point-sampled rather than area-weighted over crop masks; precipitation controls are available for wheat only; Aqua MODIS is not included; temperature specifications are linear.

---

## Citation

If you use this code or the derived panel, please cite the paper:

```
[Author], Satellite-Informed Climate Analysis of Asian Crop Yield Vulnerability:
Space-Enabled Pathways for Agricultural Resilience, IAC-26-[CODE],
77th International Astronautical Congress, Antalya, Türkiye, 2026, 5–9 October.
```

Please also cite the underlying data products individually — see the paper's reference list for DOIs.

---

## Licence

Code released under the MIT Licence (see `LICENSE`).

Derived data in `data/processed/` are provided for verification. The underlying source datasets remain subject to the terms of their respective providers; all are open-access at time of publication.
