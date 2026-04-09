# Demographic and Geographic Patterns in Policing in New Orleans

SDS 357 — Cook the Data Cajun Way

----------

## Project Description

This project aims to analyze potential racial and geographic disparities in police stops in New Orleans using data from the Stanford Open Policing Portal (SOPP) combined with U.S. Census ACS data  in the form of 5 year estimates. The analysis covers 2011–2015 and tries to understand how stop outcomes, such as warnings, citations, arrests, relate to driver demographics, neighborhood characteristics, and location.

Some of our motivating questions include:

-   Are demographic and socioeconomic factors associated with stop outcomes?
-   Do warning, citation, and arrest rates look different across different parts of New Orleans?
-   Does Mardi Gras affect how often stops happen and what comes of them?

----------

## Data

### Main Dataset: Stanford Open Policing Project

-   Download from [https://openpolicing.stanford.edu/](https://openpolicing.stanford.edu/). Navigate to the Louisiana/New Orleans sections and download the .csv file. 
-   This data contains individual police stop records for New Orleans. We filtered to 2011–2015, and ended up with ~173k observations after cleaning
-   Key variables: stop date/time, driver age/sex/race, lat/lng, reason for stop, search indicators, outcome

### Supplementary Dataset: ACS 5-Year Estimates (2011- 2015)

-  The source is the US Census Bureau
- This dataset contains census tract-level income, race, and education data for New Orleans tracts
- The files for these estimates can be found in the files in `data/raw/acs/'

### Supplementary Dataset: Census Tract Shapefile

-  The source is the US Census Bureau
-  It is used for spatial join between stop coordinates and census tracts, covers census tracts in New Orleans
-  Files in `data/raw/shapefiles/`. There are many of them, but they generally follow this pattern.

----------

## Current Repo Structure

(This may be subject to change)

```
sds357-project-sp26-cook-the-data-cajun-way/
│
├── data/
│   ├── raw/
│   │   ├── acs/                        # ACS census data
│   │   │   ├── 2015DEM.csv
│   │   │   ├── 2015EDU.csv
│   │   │   └── 2015INC.csv
│   │   ├── shapefiles/                 # Census tract shapefiles
│   │   │   └── tl_2015_22_tract.*
│   │   └── policing_imputed_coords.csv.gz
│   └── processed/
│       ├── stops_clean.csv.gz          # Final merged dataset
│       └── geocoded_locations.csv
│
├── notebooks/
│   ├── geocoding.ipynb                 # Geocoding code
│   ├── mergeclean.ipynb                # Merges ACS, SOPP data
│   ├── eda.ipynb                       # EDA figures
│   └── modeling.ipynb                  # Models
│
├── README.md
└── requirements.txt
```

----------
## Setup

You may need Python 3.8+ and Jupyter.

```bash
#1. Clone the repo
git clone git@github.com:utexas-SDS-357/sds357-project-sp26-cook-the-data-cajun-way.git
cd sds357-project-sp26-cook-the-data-cajun-way

#2. (Optional) You might want to create a virtual environment for this
python -m venv venv
source venv/bin/activate  # Mac/Linux — use venv\Scripts\activate on Windows I think

# 3. Install dependencies through requirements.txt
pip install -r requirements.txt
# Or you could install them manually 
pip install pandas numpy geopandas geopy scikit-learn statsmodels matplotlib seaborn tqdm shapely

```

----------

## Workflow and Usage, Reproduction

The full pipeline can be run in several steps. You might only need to rerun steps 1 and 2 if you are reproducing from scratch, since their outputs are already in the repo in their respective folders.


**1. Geocoding** (`notebooks/geocoding.ipynb`)

Description: Many stops in the SOPP dataset are missing lat/long data, so this step is to impute coordinates for locations without coordinates by using addresses, if the observations have any. Running this notebook (`geocoding.ipynb`) attempts to impute coordinates using addresses for observations that may not have lat/long data already.

Input:  Its input is the raw SOPP csv file (which you may have locally).

Output: Its output are two files. Firstly it produces `data/processed/geocoded_locations.csv`, which is like a lookup table that maps address strings to imputed lat/lng coordinates, and also `data/raw/policing_imputed_coords.csv.gz`, which is the full SOPP dataset that has `final_lat` and `final_lng` columns which represent original coordinates when available, and imputed coordinates elsewhere mostly. 

Note that this notebook can take 1+ hours to run fuly due to the Nominatim API rate limits, and the output file is already in the repo.

**2. Merge** (`notebooks/mergeclean.ipynb`)

Description: The purpose of this step is to read in the geocoded SOPP data, filter it from 2011-2015, clean the ACS datasets, and join stops to census tracts so that each stop can get associated with neighborhood-level socioeconomic characteristics.

Input: `data/raw/policing_imputed_coords.csv.gz` (SOPP data with imputed coordinates),  `data/raw/acs/2015DEM.csv` ( ACS demographic data), `data/raw/acs/2015EDU.csv`  (ACS education data), `data/raw/acs/2015INC.csv` (ACS income data), `data/raw/shapefiles/tl_2015_22_tract.shp` (Census tract shapefiles).

Output:  Produces `data/processed/stops_clean.csv.gz` as the output, which represents the final merged dataset containing 274657 rows and 77 columns, and includes many key variables. 

Note that the stops_clean.csv.gz file is already in the repo.


**3. EDA** (`notebooks/eda.ipynb`)

Description: The purpose of this is to do EDA of stop patterns across demographic groups, time, and geography. 

Input: `data/processed/stops_clean.csv.gz`, `data/raw/shapefiles/tl_2015_22_tract.shp` (Census tract shapefiles for spatial mapping)

Output:  Several figures that display information about the data, may be available in the notebook.

**4. Modeling** (`notebooks/modeling.ipynb`)

Description:  The purpose of this step is to predict and analyze stop outcomes through different modeling approaches. It currently includes 3 unique approaches.

Binary Random Forest Classifier: The first model is a binary Random Forest that predicts arrest vs not arrest. It uses a tract-aware train test split, and includes features such as age, race, sex, reason for stop, time variables, ACS variables. 

	 - Input: `data/processed/stops_clean.csv.gz`.
	 - Output:  model, confusion matrix, precision/recall scores, ROC AUC, and feature importance rankings.
	
Multiclass Random Forest model: Aims to predict warning/citation/arrest/none outcomes, which has a similar setup to the binary model and outputs a confusion matrix, classification report, and feature importances.

	 - Input:  `data/processed/stops_clean.csv.gz`.
	 - Output: confusion matrix, classification report, and feature importances.

Multinomial Logistic Regression model:  Aims to predict stop outcome using features such as age, race, sex, lat/lng, and other temporal features.

	 - Input: `data/processed/stops_clean.csv.gz`.
	 - Output: classification report containing information regarding precision/recall per outcome class.

All models read input from `data/processed/stops_clean.csv.gz`.

----------

## Dependencies


| Category |Packages| Purpose|
|---|---|---|
| Data | pandas, numpy | Data cleaning, handling, manipulation, etc. |
| Spatial | geopandas, shapely| Working with shapefiles, coordinates, spatial information, etc. |
| Geocoding | geopy, tqdm| Working with Nominatim API, tracking progress |
| Modeling | scikit-learn, statsmodels | Working with models |
| Visualization|matplotlib, seaborn |Generating visualizations | 






[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/MFzEnxem)
