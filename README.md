# SBD1ILV-Statistics-and-Big-Data-Analysis

## Forecasting Influenza Waves in the EU

SBD group project – topic no. 5: *Influenza and respiratory viruses in the EU*

**Scope: influenza only.** The sources also contain other respiratory viruses (RSV, SARS-CoV-2) and acute respiratory infections (ARI), but these are not part of this project. The final joined dataset keeps only influenza indicators, ILI and SARI rates, and all-cause deaths.

## About the topic

Every winter an influenza wave puts pressure on GP practices and hospitals. Its onset, peak and intensity differ from season to season. Weekly influenza surveillance data from European and global sources could be used, for example, to:

- estimate **when** a wave starts and when it peaks,
- forecast **how strong** it will be (cases per 100,000 population, test positivity),
- estimate the **hospital burden** (SARI, hospitalisations, ICU admissions),
- compare **Slovakia** with the rest of the **EU**,
- link influenza waves to **excess mortality**.

## Datasets

| | ECDC ERVISS | WHO FluNet | Eurostat demo_r_mwk_20 |
|---|---|---|---|
| **Content** | Weekly respiratory virus surveillance: ILI/ARI in primary care, SARI in hospitals, tests and positivity (influenza, RSV, SARS-CoV-2), hospitalisations, ICU, deaths, influenza subtypes – **we use only the influenza part** | Laboratory-confirmed influenza specimens by type and subtype (A/H1N1, A/H3, B/Victoria…) and number of specimens tested | Weekly deaths by sex and 20-year age group |
| **Source** | [github.com/EU-ECDC/Respiratory_viruses_weekly_data](https://github.com/EU-ECDC/Respiratory_viruses_weekly_data), methodology at [erviss.org](https://erviss.org) | [who.int/tools/flunet](https://www.who.int/tools/flunet) | [Eurostat data browser](https://ec.europa.eu/eurostat/databrowser/view/demo_r_mwk_20/default/table) |
| **Period** | 2022-W25 → present | since 1995 | since 2000 (varies by country) |
| **Countries** | 28–31 EU/EEA countries incl. SK | worldwide | EU + EFTA and others (39) |
| **Format** | 9 CSV files in long format (~27 MB) | 1 large CSV + data dictionary | CSV (SDMX) |
| **License** | EUPL-1.2 (repository); ECDC data may be reused with attribution | [WHO data policy](https://www.who.int/about/policies/publishing/data-policy) – free non-commercial use with attribution | Eurostat – free reuse with attribution (CC BY 4.0) |

### Why these datasets

- **ECDC ERVISS** is the main source. It is the official European surveillance with harmonised indicators for all countries, so SK can be compared directly with the EU. It also covers hospital burden (SARI, ICU). Its drawback is a short history (since 2022).
- **WHO FluNet** adds a **long history** (decades of seasons, including pre-COVID years) and **influenza subtypes**. Without it a seasonal model would have only ~4 ECDC seasons to learn from. Seasons 2020/21 and 2021/22 are atypical (almost no influenza during COVID-19).
- **Eurostat deaths** is an independent source that allows measuring **excess mortality** during waves, i.e. the real population impact of influenza, especially for people aged 60+.

### How we will combine them

All three sources share **country** and **ISO week**, so the join key is `(country, ISO year-week)`.

| Source | Country | Week |
|---|---|---|
| ECDC | `countryname` (e.g. `Slovakia`) | `yearweek` (`2024-W05`) |
| WHO FluNet | `COUNTRY_CODE` (ISO3, `SVK`) | `ISO_YEAR` + `ISO_WEEK`, `ISO_WEEKSTARTDATE` |
| Eurostat | `geo` (ISO2, `SK`) | `TIME_PERIOD` (`2024-W05`) |

Steps:

1. **Keep only influenza:** from ECDC keep ILI and SARI rates and all influenza indicators; drop RSV, SARS-CoV-2 and ARI.
2. **Harmonise countries** to ISO2 codes using a lookup table (`Slovakia` → `SK`, `SVK` → `SK`).
3. **Harmonise weeks** to the Monday date of the ISO week.
4. **Reshape ECDC data** from long to wide format (one column = indicator × age group or influenza subtype).
5. **Aggregate WHO rows** for the same country and week (there can be several by source type) and drop Eurostat week `W99` (unknown week).
6. **Join everything** with a `left join` onto a *country × week* table. Before 2022 the ECDC columns stay empty.
7. **Compute expected mortality** (mean of the same week in 2015–2019) and **excess mortality** from the deaths data.

**Things to watch out for:**

- ECDC and Eurostat **revise data retrospectively** (the latest Eurostat weeks are provisional, flag `p`). Always record the download date.
- Case definitions and reporting practices differ between countries. Compare absolute numbers with care and prefer trends and positivity.
- Some ECDC series for Slovakia have gaps (`SARITestsDetectionsPositivity`, `activityFluTypeSubtype`). `ILIARIRates` and `sentinelTestsDetectionsPositivity` are complete.

## Download

Automatic: run the notebooks `1_ECDC_ERVISS.ipynb`, `2_WHO_FluNet.ipynb` and `3_Eurostat_umrtia.ipynb`. They download the data into the `raw_data/` folder.

Manual: save the downloaded files into the **`raw_data/`** folder and write the download date into `raw_data/DOWNLOADED.txt`.

### 1. ECDC ERVISS

1. Open <https://github.com/EU-ECDC/Respiratory_viruses_weekly_data>.
2. Click the green **Code → Download ZIP** button and unzip the archive.
3. Copy the 9 `*.csv` files from the `data/` folder into `raw_data/`. The `data/snapshots/` folder holds older versions and is not needed.

Alternatively from the terminal: `git clone --depth 1 https://github.com/EU-ECDC/Respiratory_viruses_weekly_data.git`

### 2. WHO FluNet

1. Open <https://www.who.int/tools/flunet>.
2. In the **Download the full dataset** section click **FluNet dataset (CSV)**. The file is large, so the download takes a while.
3. On the same page also download the **Data dictionary (CSV)**, which explains the columns.
4. Save both files into `raw_data/`.

If you only need a subset (e.g. SK or the EU), use the **View/download filtered data** link. It opens Power BI, where you can set filters and export the data.

### 3. Eurostat – weekly deaths

1. Open <https://ec.europa.eu/eurostat/databrowser/view/demo_r_mwk_20/default/table>.
2. Under **Customise your dataset** select the countries (at least SK and the EU countries you want to compare), all age groups, sex *Total, Males, Females* and the period from 2015.
3. Click **Download**, choose **CSV** and *Full dataset* (or *Data as displayed* if you filtered).
4. Save the file into `raw_data/`.