# Calgary Traffic Incidents Dashboard

A single-page Power BI dashboard on the socioeconomic and geographic factors behind traffic incidents in Calgary: where incidents happen, how they relate to neighbourhood income, and when they occur (time of day, day of week and month).

Data: City of Calgary Open Data. Traffic Incidents, Calgary Equity Index Overlay Statistics (2021 Census) and Community District Boundaries.

## Folder layout

```
calgary-traffic-dashboard/
├── data/
│   ├── Traffic_Incidents.csv                        incidents with CSA (AREA_ID) and community (COMM_CODE)
│   ├── Calgary_Equity_Index_Overlay_Statistics.csv  one row per CSA: population, income, equity measures
│   ├── Community_District_Boundaries.csv            one row per community: class, era, area (km²)
│   ├── calgary_csa_topo.json                        CSA boundaries for the Shape Map
│   └── calgary_community_topo.json                  community boundaries for the Shape Map
└── report/
    ├── Calgary Traffic Dashboard.pbip               open this file to work
    ├── Calgary Traffic Dashboard.Report/
    └── Calgary Traffic Dashboard.SemanticModel/
```

## One-time setup

1. Install Git and an up-to-date Power BI Desktop.
2. Clone the repo to exactly this location, so the data paths work without changes:
   ```
   cd C:\Repos
   git clone https://github.com/AdrianJulve/calgary-traffic-dashboard.git
   ```
3. In Power BI Desktop: File → Options and settings → Options → Preview features → tick **Shape map visual**, then restart Power BI.
4. Open `report\Calgary Traffic Dashboard.pbip` and click **Refresh** once to load the data. No Python is needed.

If you must clone somewhere else, change the `DataFolder` parameter (Transform data → Manage Parameters) to your `data\` folder, and don't commit that change.

## Working on the dashboard

1. Close Power BI, then run `git pull`.
2. Open the `.pbip`, make your changes, then save and close Power BI.
3. Commit and push from the repo's top folder:
   ```
   git add .
   git commit -m "Describe what you changed"
   git push
   ```

Rules:
- Always close Power BI before pulling.
- Tell the group which visual or measure you're working on. Two people editing the same visual or measure at once causes merge conflicts.
- Never use `git push --force`.
- If a push is rejected, run `git pull` first, then push again.

## Data model

Tables:
- **Incidents** (fact): one row per incident
- **Equity**: one row per community service area (CSA)
- **Communities**: one row per community
- **Bivariate_Legend**: the 9 map colour classes

Relationships (all many-to-one, single direction):
- `Incidents[AREA_ID]` → `Equity[AREA_ID]`
- `Incidents[COMM_CODE]` → `Communities[COMM_CODE]`
- `Equity[Bivariate_Class]` → `Bivariate_Legend[Class]` (one-to-many from the legend)

Don't rename tables or the columns above. The measures, calculated columns and map colours depend on them.

Key measures:

| Measure | Purpose |
|---|---|
| `Incident Count` | count of incidents in the current filter |
| `Rate per 1k` | incidents per 1,000 residents per year |
| `Income Gap` | rate in the lowest-income third ÷ rate in the highest-income third |
| `Peak Hour` | busiest hour of the day |
| `Monthly Rate per 100k` | incidents per 100,000 residents per month, adjusted for days of data |
| `CSA Tooltip` | text for the map's hover card |
| `Map Subtitle` | map subtitle that changes with the legend selection |

## Method notes

- **Map colours** are fixed to 2017–2025 and use local and arterial roads in residential communities only (`Equity[Local_Rate]`). Slicers don't change them; this is intentional.
- **Precomputed columns.** `AREA_ID` and `COMM_CODE` on incidents, and `Area_km2`, were computed outside Power BI by matching each incident's coordinates to the boundary polygons. Power BI can't do point-in-polygon matching. 785 incidents fall outside every CSA and 65 outside every community; these have blank IDs.
- **Major corridors** (Deerfoot, Glenmore, Stoney, Crowchild, Sarcee and similar) are identified by street name in `Road_Class`.
- **Gaps in the source data:** January 2017, June–August 2019 and June 2025 are missing or nearly empty. Monthly rates are blanked when a month has fewer than 15 days of data.
- **Population and income** come from the 2021 Census via the Calgary Equity Index. Rates use residential population, so areas with many commuters or industrial land (e.g. downtown) read high.
