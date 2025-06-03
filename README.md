# wa‑ev‑geo‑temporal‑insights 
Geospatial & time‑series analysis of Washington EV registrations

## Table of Contents

* [Business Context and Objectives](#business-context-and-objectives)
* [Hypotheses](#hypotheses)
* [Project Scope](#project-scope)
* [Repository Structure](#repository-structure)
* [Data & Methodology](#data--methodology)

  * [Environment and Dependencies](#environment-and-dependencies)
  * [Data Pipeline](#data-pipeline)
  * [Clustering Methodology](#clustering-methodology)
  * [Cluster Interpretation](#cluster-interpretation)
* [Key Findings](#key-findings)

  * [Cluster Profiles](#cluster-profiles)
  * [Top Models and Makes by Cluster](#top-models-and-makes-by-cluster)
  * [Geographic Insights](#geographic-insights)
* [Limitations](#limitations)
* [Data Sources and References](#data-sources-and-references)

## Business Context and Objectives

Washington State aims to accelerate electric vehicle (EV) adoption to advance sustainable transportation and clean‐energy goals. Seattle’s tech‐driven environment makes it a bellwether for statewide trends. This project uses licensing and census data to:

1. **Analyze** regional EV ownership patterns (BEVs vs. PHEVs) across Washington.
2. **Segment** EV owners via clustering to identify demographic and economic profiles.
3. **Inform** policymakers and industry stakeholders on targeted incentives, infrastructure planning, and marketing strategies.

## Hypotheses

1. **Income & Vehicle Cost:** Higher‐income households cluster around premium, recent‐model EVs; lower‐income clusters favor older or budget models.
2. **Range & Model Year:** Clusters differ by electric range and model year—some prioritize long range over the newest release, others value recency.
3. **Geographic Adoption:** Urban centers (e.g., Seattle area) exhibit higher concentrations of newer, premium EVs compared to rural counties.

## Project Scope

* **Geography:** All registered BEVs and PHEVs in Washington State (any vehicle with a WA registration, regardless of physical location).
* **Time Frame:** Latest annual snapshot (2024) from the Washington State Department of Licensing and HIFLD.
* **Entities:** 204,544 EV records, each with vehicle attributes (model, make, model year), owner demographics (income), and location (county, city, census tract).
* **Deliverables:**

  * Data preprocessing pipeline for combining numeric and categorical features.
  * K‑means clustering to segment EV owners into four clusters.
  * Visualization dashboards (Tableau) showcasing adoption patterns and cluster distributions.
  * Strategic recommendations for policy, marketing, and infrastructure.

## Repository Structure

```
/ (root)
├── data/
│   ├── raw/
│   │   ├── ev_population_data.csv        # Downloaded from data.wa.gov
│   │   ├── income_by_zip.csv             # IRS income data by ZIP code
│   │   └── car_prices.csv                # Aggregated MSRP and current prices
│   └── processed/
│       ├── ev_cleaned.csv                # Filtered and cleaned EV records
│       ├── features_matrix.npy           # Combined numeric + encoded categorical features
│       ├── cluster_labels.csv            # Assigned cluster for each EV record
│       └── cluster_centers.csv           # K‑means cluster centroids
├── notebooks/
│   ├── 1_data_preprocessing.ipynb        # Cleaning, encoding, and scaling steps
│   ├── 2_kmeans_clustering.ipynb         # K‑means elbow plot and cluster training
│   ├── 3_tsne_visualization.ipynb        # t‑SNE reduction of cluster centers
│   └── 4_cluster_analysis.ipynb          # Interpreting clusters; top makes/models
├── scripts/
│   ├── fetch_data.py                     # Retrieves EV, income, and car price datasets
│   ├── preprocess_data.py                # Merges raw data; encodes categorical features
│   ├── run_clustering.py                 # Fits K‑means, saves labels and centers
│   ├── tsne_plot.py                      # Generates t‑SNE plot of cluster centers
│   └── summarize_clusters.py             # Computes numeric summaries per cluster
├── dashboards/
│   └── tableau_workbook.twbx             # Interactive Tableau dashboards (EV adoption, clusters)
├── requirements.txt                      # Python packages (pandas, numpy, scikit‑learn, matplotlib, tsne, etc.)
├── environment.yml                       # Conda environment definition
├── README.md                             # This file
└── results/
    ├── figures/
    │   ├── elbow_plot.png                # Inertia vs. cluster count
    │   ├── tsne_clusters.png             # t‑SNE visualization of centroids
    │   ├── cluster_centers_plot.png      # Line chart: numeric features by cluster
    │   ├── top_models_clusters.png       # Bar charts: top 5 models per cluster
    │   ├── top_makes_clusters.png        # Bar charts: top 5 makes per cluster
    │   └── top_cities_clusters.png       # Bar charts: top 5 cities per cluster
    └── tables/
        ├── cluster_summary_stats.csv     # Mean and std dev of numeric features by cluster
        └── cluster_profile_description.csv # Qualitative descriptors of each cluster
```

* **data/raw:** Original downloads (EV registrations, income by ZIP, MSRP/current prices).
* **data/processed:** Cleaned and merged EV records with engineered features and clustering outputs.
* **notebooks:** Step‑by‑step exploratory analyses and visualizations.
* **scripts:** Production‐ready modules to automate ingestion, preprocessing, clustering, and summarization.
* **dashboards:** Final Tableau workbook for interactive BI presentations.
* **results:** Static outputs (figures, summary tables) for quick reference.

## Data & Methodology

### Environment and Dependencies

* **Python Version:** 3.9+
* **Key Packages:**

  * `pandas`, `numpy` (data manipulation)
  * `scikit‑learn` (`KMeans`, `StandardScaler`, `OneHotEncoder`, `TSNE`)
  * `matplotlib` (plotting clusters and elbow)
  * `seaborn` (enhanced visual styling)
  * `tableau-api-lib` (optional for Tableau automation)
* **Setup:**

  ```bash
  conda env create -f environment.yml
  conda activate ev_clustering
  # Or:
  pip install -r requirements.txt
  ```

### Data Pipeline

1. **Fetch & Load Raw Data**

   * **fetch\_data.py:**

     * Download `Electric Vehicle Population Data` CSV from data.wa.gov (DOL + HIFLD).
     * Load IRS income data by ZIP code and multiple car price sources (e.g., Kelley Blue Book, Edmunds).
2. **Merge & Clean**

   * **preprocess\_data.py:**

     * Read EV data (204,544 rows, \~110 VIN features + owner info).
     * Select relevant numeric columns:

       * `Electric Range`, `Model Year`, `Income`, `Car Price Today`, `Car Price Launched`
     * Select relevant categorical columns:

       * `County`, `City`, `Make`, `Model`, `Electric Vehicle Type`, `CAFV Eligibility`, `Electric Utility`
     * Drop rows with missing values in these columns (final \~204,544 × 12 matrix).
     * Convert categorical columns to string type.
     * One‑hot encode categorical features (`OneHotEncoder(sparse_output=False, handle_unknown='ignore')`).
     * Scale numeric features using `StandardScaler()`.
     * Horizontally stack scaled numerics and encoded categoricals → feature matrix (204,544 × 746).
     * Save `ev_cleaned.csv` and `features_matrix.npy`.
3. **Clustering Execution**

   * **run\_clustering.py:**

     * Perform K‑means clustering on the feature matrix.
     * Use an elbow plot (inertia vs. cluster count 1–10) to confirm 4 clusters (elbow at k = 4).
     * Fit `KMeans(n_clusters=4, random_state=42)`.
     * Assign cluster labels (0–3) to each record and save to `cluster_labels.csv`.
     * Extract cluster centroids (4 × 746) and save to `cluster_centers.csv`.
4. **Dimensionality Reduction & Visualization**

   * **tsne\_plot.py:**

     * Apply t‑SNE on cluster centroids (4 points) with `perplexity=3` → 2D coordinates.
     * Plot centroids in 2D space to confirm separation (save as `tsne_clusters.png`).
5. **Cluster Interpretation & Summary**

   * **summarize\_clusters.py:**

     * Load `cluster_labels.csv`, merge with `ev_cleaned.csv` to associate features.
     * Compute mean (and standardized) values of numeric features by cluster:

       * `Electric Range`, `Model Year`, `Income`, `Car Price Today`, `Car Price Launched`.
     * Describe each cluster qualitatively (e.g., “higher‑range older EVs,” “newer premium EVs”).
     * Extract top 5 models, makes, and cities by count in each cluster.
     * Save `cluster_summary_stats.csv` and `cluster_profile_description.csv`.

### Clustering Methodology

* **Choosing K:**

  * Construct elbow plot of inertia for k = 1…10.
  * Identify the elbow at k = 4 (diminishing returns beyond 4 clusters).
* **Feature Matrix:**

  * Combined 5 scaled numeric features and 741 one‑hot encoded categorical features → 746 total dimensions.
* **K‑Means Parameters:**

  * `n_clusters=4`, `init='k-means++'`, `random_state=42`.
  * Converges in \~ 10–15 iterations (based on inertia).
* **t‑SNE for Visualization:**

  * Reduce 746D centroids to 2D using `TSNE(n_components=2, random_state=42, perplexity=3)`.
  * Confirm that clusters remain well‐separated in 2D.

### Cluster Interpretation

After clustering, numeric cluster centers (standardized values) reveal distinct profiles:

* **Cluster 0:**

  * **Electric Range:** +2.11 (above average)
  * **Model Year:** –0.87 (older vehicles)
  * **Income:** +0.07 (slightly above average)
  * **Car Price Today:** –0.41 (below average)
  * **Car Price Launched:** +0.32 (above average)
    **Profile:** Owners prioritizing long range with older, moderately priced EVs; likely early adopters with mid‑income.
* **Cluster 1:**

  * **Electric Range:** –0.53 (below average)
  * **Model Year:** +0.57 (newer vehicles)
  * **Income:** –0.008 (nearly average income)
  * **Car Price Today:** +0.11 (slightly above average)
  * **Car Price Launched:** –0.20 (below average)
    **Profile:** Mid‑income owners choosing newer models with moderate range and average pricing; likely value‑conscious early adopters.
* **Cluster 2:**

  * **Electric Range:** +0.09 (near average)
  * **Model Year:** –1.60 (significantly older)
  * **Income:** –0.17 (below average)
  * **Car Price Today:** –1.23 (significantly below average)
  * **Car Price Launched:** –0.86 (below average)
    **Profile:** Budget‑conscious owners of older EVs with moderate range; likely leveraging incentives or used market.
* **Cluster 3:**

  * **Electric Range:** –0.53 (below average)
  * **Model Year:** +0.67 (newer vehicles)
  * **Income:** +0.21 (above average)
  * **Car Price Today:** +2.09 (significantly above average)
  * **Car Price Launched:** +2.11 (significantly above average)
    **Profile:** Affluent owners purchasing premium, recent EVs (lower range but higher price points), likely prioritizing status and technology.

## Key Findings

### Cluster Profiles

1. **Cluster 0 (119,005 records):**

   * **Description:** Long‐range older EVs at moderate current prices.
   * **Demographics:** Slightly above‑average income; older model years.
   * **Implication:** Target infrastructure for longer‑range support; promote refurbishment incentives for older units.
2. **Cluster 1 (33,410 records):**

   * **Description:** Newer, affordable EVs with moderate range.
   * **Demographics:** Near‑average income, focus on recent technology.
   * **Implication:** Highlight tax credits for new buyers; emphasize urban charging stations.
3. **Cluster 2 (32,519 records):**

   * **Description:** Budget EVs—older models with modest range.
   * **Demographics:** Below‑average income; cost‑sensitive owners.
   * **Implication:** Recommend secondhand EV programs, low‑cost charging solutions.
4. **Cluster 3 (19,610 records):**

   * **Description:** Premium EVs—recent models costing well above average, shorter range.
   * **Demographics:** Higher income; luxury and early‑adopters.
   * **Implication:** Suggest high‐performance charging networks; support upscale incentives (e.g., HOV access).

> ![Elbow Plot](results/figures/elbow_plot.png)
> *Inertia vs. cluster count identifies ideal k = 4.*
>
> ![t-SNE Clusters](results/figures/tsne_clusters.png)
> *t‑SNE visualization confirms distinct separation of 4 cluster centroids.*
>
> ![Cluster Centers](results/figures/cluster_centers_plot.png)
> *Standardized numeric feature values by cluster center.*

### Top Models and Makes by Cluster

* **Cluster 0 (Long‑Range Older EVs):**

  * **Top 5 Models:** Tesla Model S, Tesla Model 3, Nissan Leaf, Chevrolet Bolt EV, BMW i3.
  * **Top 5 Makes:** Tesla, Nissan, Chevrolet, BMW, Ford.
* **Cluster 1 (Newer Affordable EVs):**

  * **Top 5 Models:** Tesla Model 3, Ford Mustang Mach‑E, Kia Niro EV, Chevrolet Bolt EV, Nissan Leaf.
  * **Top 5 Makes:** Tesla, Ford, Kia, Chevrolet, Nissan.
* **Cluster 2 (Budget Older EVs):**

  * **Top 5 Models:** Nissan Leaf, Chevrolet Volt (used), Ford Focus Electric, BMW i3, Toyota Prius PHEV.
  * **Top 5 Makes:** Nissan, Chevrolet, Ford, BMW, Toyota.
* **Cluster 3 (Premium New EVs):**

  * **Top 5 Models:** Tesla Model X, Tesla Model Y, Rivian R1S, Lucid Air, Porsche Taycan.
  * **Top 5 Makes:** Tesla, Rivian, Lucid, Porsche, Mercedes‑Benz.

> ![Top Models per Cluster](results/figures/top_models_clusters.png)
> *Bar charts showing top 5 EV models in each cluster.*
>
> ![Top Makes per Cluster](results/figures/top_makes_clusters.png)
> *Bar charts showing top 5 manufacturers in each cluster.*

### Geographic Insights

* **Cluster 0 & 1 (Concentrated in Urban/Suburban Areas):**

  * Major cities: Seattle, Bellevue, Tacoma.
  * Indicates strong adoption of mid‑to‑high‑range EVs in tech hubs and supply chain corridors.
* **Cluster 2 (Scattered in Rural/Lower‑Income Counties):**

  * Counties: Yakima, Spokane, Wenatchee.
  * Reflects budget‑conscious buyers in areas with fewer charging stations; need for rural infrastructure.
* **Cluster 3 (High‑Income Enclaves):**

  * Cities: Mercer Island, Medina, Sammamish.
  * Demonstrates premium EV demand in affluent suburbs; potential for premium fast‑charging stations.

> ![Top Cities per Cluster](results/figures/top_cities_clusters.png)
> *Bar charts showing top 5 cities for each cluster.*

## Limitations

* **Annual Snapshot:** Data reflects a single 2024 release; year‑over‑year trends are not analyzed.
* **Self‑Reported Income by ZIP:** Income may not precisely reflect individual household earnings.
* **Census Tract Aggregation:** Some analyses aggregate by tract or county, obscuring micro‑local patterns.
* **Vehicle Location vs. Registration:** Vehicles registered in WA may be located out‑of‑state, slightly skewing geographic insights.
* **Feature Set Bias:** Only selected numeric and categorical features used; other EV attributes (e.g., battery size) are omitted.

## Data Sources and References

* **Washington State Electric Vehicle Population Data** (DOL + HIFLD):

  * CSV download: `https://data.wa.gov/api/views/f6w7q2d2/rows.csv?accessType=DOWNLOAD`
  * Key fields: `vin_1_10`, `county`, `city`, `state`, `zip_code`, `model_year`, `make`, `model`, `ev_type`, `cafv_type`, `electric_range`, `base_msrp`, `legislative_district`, `geocoded_column`, `electric_utility`, `2020_census_tract`, `Car Price Today`, `Car Price Launched`, `Income` (by ZIP).
* **IRS Income Data by ZIP Code (2021):**

  * `https://www.irs.gov/statistics/soi-tax-stats-individual-income-tax-statistics-2021-zip-code-data-soi-0`
* **Car Price References:**

  * Kelley Blue Book (`kbb.com`), Edmunds (`edmunds.com`), Car and Driver (`caranddriver.com`), Official brand portals, Google Search for MSRP and current prices.
* **Clustering & Visualization:**

  * Scikit‑learn documentation for `KMeans`, `StandardScaler`, `OneHotEncoder`, `TSNE`.
  * Matplotlib documentation for plotting routines.
