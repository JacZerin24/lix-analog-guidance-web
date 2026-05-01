# WFO LIX Experimental Analog Severe Weather Guidance

This repository hosts a public static web viewer for an experimental analog-based severe weather guidance system focused on the NWS New Orleans/Baton Rouge County Warning Area, WFO LIX.

The viewer displays gridded analog probabilities for severe-weather hazards using RRFS forecast data compared against a historical ERA5 analog library. It is designed for situational awareness, internal research, and experimental pattern-recognition guidance.

## Live Viewer

GitHub Pages URL:

```text
https://YOUR-USERNAME.github.io/lix-analog-guidance-web/
```

Replace `YOUR-USERNAME` with the GitHub account or organization that owns this repository.

## Important Disclaimer

This is experimental analog guidance.

It is not official NWS forecast guidance, not calibrated SPC-style probabilistic guidance, and not a replacement for operational forecast reasoning. The probabilities shown on the map are nearest-neighbor historical analog frequencies, not official forecast probabilities.

Use this tool as supplemental situational awareness and pattern-recognition guidance only.

## What the System Does

In simple terms, the system asks:

> Which historical ERA5 environments looked most similar to the current RRFS forecast environment, and what severe weather reports occurred with those historical analogs?

For each forecast grid box, the system:

1. Reads RRFS pressure-level forecast fields.
2. Converts the forecast environment into the same feature format as the historical ERA5 analog library.
3. Finds the nearest historical ERA5 analogs in standardized feature space.
4. Counts how many of those nearest analogs had nearby severe-weather reports.
5. Converts those counts into analog-frequency probabilities.
6. Displays the results on an interactive web map.
7. Allows users to click a grid cell to inspect the top analogs, feature similarities, and report-day assets.

## Hazards Displayed

The viewer can display:

- Any severe
- Tornado
- Severe hail
- Damaging wind
- Significant tornado
- Significant hail
- Significant damaging wind

## Current Feature Set

The current clean baseline analog library uses pressure-level fields only:

- `u` wind
- `v` wind
- `z_gpm` geopotential height
- `r` relative humidity

at:

- 250 hPa
- 500 hPa
- 700 hPa
- 850 hPa

This means the current analog matching is based on upper-air wind, height, and relative humidity patterns.

It does not currently include CAPE, CIN, lapse rates, SRH, effective shear, surface dewpoint, or storm-scale reflectivity fields unless those are added in a future library version.

## What the Probabilities Mean

The probabilities are analog frequencies.

For example, if a grid cell shows:

```text
Severe Hail: 25%
```

and the system is using 40 nearest analogs, that means:

```text
10 of the 40 nearest historical analogs had a severe hail label.
```

It does not mean there is a calibrated 25% chance of severe hail in the official forecast sense.

## Analog Distance

Each grid cell includes analog distance diagnostics. Lower distance means the current RRFS forecast environment more closely resembles historical ERA5 analogs.

Useful interpretation:

- Lower distance: stronger historical match
- Higher distance: weaker historical match
- High probability with low distance: more confidence in the analog signal
- High probability with high distance: use caution, because the system may be stretching to find analogs
- Low probability with high distance: use caution, because the forecast environment may be outside the library’s comfort zone

Approximate practical guide:

| Mean Analog Distance | Interpretation |
|---:|---|
| `< 1.75` | Good match |
| `1.75 to 2.50` | Fair match |
| `2.50 to 3.25` | Marginal match |
| `> 3.25` | Poor match |

These thresholds are preliminary and should be refined through verification.

## Feature-Level Explanation

When available, the clicked grid-cell panel shows why the analogs matched.

Feature explanations are based on standardized differences between the forecast feature vector and the historical analog feature vectors.

The viewer can show:

- Top similar features
- Top different features
- Feature details for individual top analogs

Smaller standardized differences mean the forecast and analog were more alike for that field.

These explanations only apply to the fields currently included in the analog library.

## Analog-Day Reports

The viewer can also display severe reports from analog days.

There are two different report concepts:

### Report-Day Assets

These show reports that occurred on the analog local calendar day.

They answer:

> What severe reports occurred on that analog day?

### Matched Reports

These are stricter. They show reports that were close to the specific analog grid point and valid time, using the configured time window and radius.

They answer:

> What reports matched this specific analog sample?

A null analog can have no matched reports even if the broader analog day had severe weather elsewhere.

## Repository Contents

This GitHub Pages repository should contain only public, web-safe static files.

Typical contents:

```text
index.html
latest.json
web_latest.json
cycles/
  YYYYMMDDHH/
    plots/
    webdata/
    netcdf/
      *_analogs.json
```

The repository should not contain:

```text
RRFS GRIB2 files
ERA5 NetCDF files
probability NetCDF files
analog library .npz files
model_cache/
era5_cache/
large logs
temporary cfgrib index files
```

## How the Website Is Updated

The operational workflow runs on a dedicated machine. The machine generates the latest static website assets, then commits and pushes only the web-safe output to this GitHub Pages repository.

The normal workflow is:

```text
06_run_lix_analog_cycle.py
  finds latest RRFS cycle
  downloads/reuses model files
  scores forecast grids against the ERA5 analog library
  writes probability NetCDFs
  writes top analog JSON files
  creates static PNG maps
  exports interactive GeoJSON and web_latest.json

08_build_analog_report_assets.py
  reads analog JSON files
  reads tornado/hail/wind report CSVs
  creates analog-day report GeoJSON/PNG files
  creates matched_reports_index.json

Batch job
  copies web-safe files into this GitHub Pages repo
  excludes heavy files like .nc, .grib2, .npz
  commits and pushes the updated website
```

## Local Operational Folder

On the dedicated machine, the working folder is expected to be something like:

```text
L:\Users\Zeringue\LIXAnalog
```

The GitHub Pages repo is cloned into:

```text
L:\Users\Zeringue\LIXAnalog\github_pages
```

The generated local website output is:

```text
L:\Users\Zeringue\LIXAnalog\web_lix_analog
```

Only the web-safe output is copied into `github_pages`.

## GitHub Pages Publishing

This repo is published with GitHub Pages from:

```text
Branch: main
Folder: /root
```

The site entry point is:

```text
index.html
```

An empty `.nojekyll` file is included so GitHub Pages serves the static files directly.

## Suggested `.gitignore`

This repository should include a `.gitignore` like:

```gitignore
*.nc
*.grib2
*.idx
*.tmp
*.log
*.npz

model_cache/
era5_cache/
__pycache__/
.ipynb_checkpoints/
scratch/
temp/
```

## Data Sources

This project uses:

- Historical ERA5 reanalysis fields
- Historical tornado, hail, and damaging wind report datasets
- RRFS forecast pressure-level GRIB2 files
- County/parish and CWA shapefiles for map context

The public GitHub Pages repository does not need to include the raw model or reanalysis data.

## Viewer Features

The web viewer includes:

- Calendar-day tabs
- Valid-time selector
- Hazard selector
- Interactive Leaflet map
- Hover probability readout
- Clickable grid-cell detail panel
- Top analog list
- Feature-level analog explanation
- Analog-day report map links
- Matched-report details when available
- Experimental guidance caveats

## Known Limitations

Current limitations include:

- Probabilities are raw analog frequencies, not calibrated probabilities.
- The analog library is currently pressure-level-only.
- Negative samples may include event-day spatial nulls, not only true quiet-weather days.
- Historical reports have spatial, timing, and reporting-quality uncertainties.
- Analog distance thresholds are preliminary.
- The system should be verified locally before being used for any operational decision support.

## Recommended Use

Use this viewer to support:

- Pattern recognition
- Severe-weather situational awareness
- Comparing current forecast environments to historical analogs
- Exploring past report outcomes from similar setups
- Internal research and training

Do not use this viewer as standalone official forecast guidance.

## Project Status

Experimental prototype.

The system currently supports an end-to-end workflow from latest RRFS model run to public static web viewer.
