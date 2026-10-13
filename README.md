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

## Setup (once)

Run all `git` commands in **PowerShell**. To open it in a folder, type `powershell` in File Explorer's address bar and press Enter.

1. Install [Git](https://git-scm.com) and an up-to-date Power BI Desktop. Check Git with `git --version`.
2. Accept the GitHub invitation (email or github.com/notifications).
3. In PowerShell (Terminal):
   ```
   git config --global user.name "Your Name"
   git config --global user.email "you@example.com"
   mkdir C:\Repos
   cd C:\Repos
   git clone https://github.com/AdrianJulve/calgary-traffic-dashboard.git
   ```
   Sign in with your browser when prompted; GitHub doesn't accept passwords in the terminal.
4. In Power BI: Options → Preview features → tick **Shape map visual**, then restart (if not already turned on).
5. Open `report\Calgary Traffic Dashboard.pbip` and click **Refresh** once.

Cloned somewhere other than `C:\Repos`? Update the `DataFolder` parameter to your `data\` folder, and don't commit that change.

## Workflow

From `C:\Repos\calgary-traffic-dashboard` in PowerShell:

```
git pull                          # before opening Power BI
git add .                         # after saving and closing Power BI
git commit -m "What you changed"
git push
```

`git status` should then say `up to date with 'origin/main'`.

Rules:
- Say which visual or measure you're editing; two people on the same one causes conflicts.
- Push rejected? Run `git pull`, then push again. Never use `--force`.

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
