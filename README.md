# rwfsas

rwfsas uses public satellite and fire-perimeter data to estimate wildfire growth and improve situational awareness. It is operated from a terminal: commands and options are the inputs, progress and summary results print in the terminal, and prediction products are saved as files. No server or browser is required.

The project is a split from fireintel. Its current perimeter model is still under development; known research limitations include overestimation on small explosive fires, missing fire-type and weather context for fire-radiative-power variance, and the spatial blur of GOES observations.

## Install

Use Python 3.10 or newer. From the repository directory, install the listed dependencies:

```text
python -m pip install -r requirements.txt
```

Some features need internet access to WFIGS, GOES, VIIRS/FIRMS, and basemap services. Set `FIRMS_MAP_KEY` to a NASA FIRMS map key to enable FIRMS access. `OPENSKY_CLIENT_ID` and `OPENSKY_CLIENT_SECRET` are optional. The Synoptic API key in `sim/config.json` is optional.

## Terminal commands

Run all commands from the repository directory:

```text
python rwfsas.py --help
python rwfsas.py fires -n 10
python rwfsas.py fires --all --min-acres 1000 -n 10
python rwfsas.py run "Sisi"
python rwfsas.py run "Sisi" --lat 48.3555 --lon -120.8244
```

`fires` lists large current WFIGS fires, hiding fires marked contained or out at 90% or greater by default. Use `--all` to include contained fires. `--min-acres` changes the minimum mapped size and `-n` sets the maximum number shown.

`run` looks up a current fire by name, or accepts its coordinates directly. It checks containment when WFIGS is reachable and refuses a contained or out fire; `--force` overrides that check. The prediction runs against the latest mapped perimeter and available GOES-18/19 and VIIRS detections. Coordinates must be supplied together. Use `--k 1.0` to set the model threshold scale, or `--out PATH` to choose the output folder. By default, files go under `output/` in the repository. `--open` is optional and opens the saved image using the operating system's image viewer.

Example:

```text
python rwfsas.py run "Sisi"
Sisi: found at 48.3555, -120.8244 (4,743 ac mapped)
Sisi: base 4,743 ac as-of 10-25 18:00Z, 12 dets {'VIIRS': 12}
Sisi: FIELD pred 5,120 ac  [3,645, 6,840]  (base 4,743, k=1.0)

  prediction mode          : learned_field_v3
  first perimeter (mapped) :     4,743 ac   as-of 2026-10-25 18:00Z
  predicted now            :     5,120 ac   80% range 3,645 - 6,840
  growth                   :      1.08x   (+377 ac)
  detections               : 12  {'VIIRS': 12}

  image: .../output/Sisi/Sisi_field.png
```

The numbers above illustrate the format; live values vary by fire and run. When there are too few inputs to build a prediction grid, the command says `persistence`, holds the perimeter at the mapped acreage, and reports that no growth estimate or prediction range is available. The JSON stores the mode as `persistence` and the range as `null`. Each successful `run` writes:

- `<name>_field.png` — static map with the mapped perimeter, predicted perimeter, available detections, and acreage range.
- `<name>_field.json` — machine-readable summary with prediction mode, base and predicted acres, range (null for persistence), coordinates, timestamp, sensor counts, and model threshold.
- `<name>_field.geojson` — predicted perimeter geometry and the same summary properties, for GIS software.
- `<name>_inputs.json` — snapshot of the mapped perimeter and detections used for this prediction.

`stages` and `stage "Fire Name" 3` list and replay historical evaluation steps. The required frozen step data is not included in this repository, so these commands report that prerequisite until the data is supplied. The live `run` command uses the included deployable model and does not need historical steps.

## Repository contents

- `fpm/` — perimeter fusion, the optional interactive Perimeter Studio program, historical stages, and aircraft, camera, and source-health inputs. The rwfsas command-line interface does not launch Perimeter Studio.
- `fire-spread-lab/` — learned perimeter-field model, GOES ABI reader, live deployment code, and VIIRS FIRMS reader.
- `sim/` — wind and fuel-moisture fusion, Rothermel model, and configuration.

The runtime models are included: `fire-spread-lab/_frozen/deployable_field_v3.pkl` and `fpm/best_seg_v3.pt`. Training data (over 100 GB), historical replay inputs, and data caches are not included. Caches and some aircraft/airport lookup data are fetched on demand.

## Results and model status

Testing results are in [v0.1 documentation.pdf](v0.1%20documentation.pdf). Newer experimental versions exist outside this repository and are not included here. The project README's release target is greater than 85% acreage coverage and 0.65 IoU on growth steps over 3x; the current model still has the limitations noted above.
