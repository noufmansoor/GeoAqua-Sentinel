# GeoAqua Sentinel

A proof of concept combining Planet Tanager hyperspectral imagery, simulated IoT measurements, and machine learning to support coastal water inspection decisions.

- Team: Alreem Ahmed Alkatheeri, Nouf Mansoor Alblooshi, Mouza Abdullah Almansoori, Marya Mohammed Alhammadi
- Country: United Arab Emirates
- Event: Arab Youth Space Hackathon 2026, 813 Challenge
- Official theme: Water quality, inland and coastal water intelligence
- Study area: Tarif coastal area, Al Dhafra, Abu Dhabi
- Status: Proof of concept

The current demonstration examines water quality indicators and abnormal simulated sensor readings. Leakage detection is a future development goal. The current model does not confirm a physical leak.

## 2. Business Use Case

The intended users are water monitoring teams, environmental authorities, and coastal facility operators.

Their main decision is where to prioritize verification, inspection, or additional water sampling.

Users currently rely on field sampling, laboratory testing, local sensors, and manual comparison of monitoring results.

GeoAqua Sentinel brings satellite indicators and sensor classification into one workflow. The demonstration produces an inspection alert when the example sensor reading is abnormal or a satellite indicator exceeds a demonstration threshold.

The proposed operational use supports inspection decisions alongside field measurements and laboratory testing.

## 3. Problem and Proposed Solution

Field sampling and sensors provide measurements at specific locations and times. They leave gaps between monitoring points, limiting assessment across a wider coastal area.

The PoC examines this monitoring challenge in the Tarif coastal area. Satellite imagery provides spatial observations across the selected scene, supporting screening beyond individual sampling locations.

Planet Tanager hyperspectral imagery supports water detection, a chlorophyll proxy, and a relative turbidity indicator.

GeoAqua Sentinel combines:

- Earth Observation imagery
- Simulated IoT water measurements
- Machine learning classification
- Satellite and IoT alert rules
- Interactive sensor input controls
- Dashboard visualization

Satellite observations depend on acquisition timing and usable image coverage. The indicators require field calibration.

The satellite indices do not directly measure pH, TDS, water flow, or underground pipe leakage.

## 4. Data Used

### Satellite Data

| Information | Details |
| --- | --- |
| Provider | Planet Labs PBC |
| Instrument | Tanager hyperspectral |
| Collection | `coastal-water-bodies` |
| Selected scene | `20250511_074311_00_4001` |
| Acquisition date | 11 May 2025 |
| Product | Surface reflectance in HDF5 format |
| Asset selection | `ortho_sr_hdf5`, otherwise `basic_sr_hdf5` |
| Quality filtering | Cloud, cirrus, and nodata masks, plus invalid negative reflectance filtering |
| Licence attribution in the source notebook | CC BY 4.0, © Planet Labs PBC |

The notebook retrieves metadata for three scenes and analyzes the first:

1. `20250511_074311_00_4001`
2. `20250223_165546_32_4001`
3. `20250406_170447_47_4001`

Collection URL:

https://www.planet.com/data/stac/tanager-core-imagery/coastal-water-bodies/collection.json

The notebook downloads satellite data during execution. Satellite files are not committed to this repository.

NDCI and the turbidity proxy are relative optical indicators. They do not represent calibrated chlorophyll concentrations or turbidity in NTU.

### Synthetic IoT Data

| Information | Details |
| --- | --- |
| Dataset | `GeoAqua_Synthetic_IoT_Data.csv` |
| Provider | GeoAqua Sentinel team |
| Acquisition date | Not applicable, simulated records |
| Processing level | Generated tabular data for model training and testing |
| Number of records | 500 |
| Normal records | 250 |
| Abnormal records | 250 |
| Random seed | 42 |
| Licence | No separate public dataset licence specified |

| Column | Description | Unit |
| --- | --- | --- |
| `pH` | Acidity or alkalinity | Dimensionless |
| `Turbidity_NTU` | Simulated turbidity | NTU |
| `Temperature_C` | Simulated temperature | °C |
| `TDS_mg_L` | Simulated total dissolved solids | mg/L |
| `Water_Flow_L_min` | Simulated flow | L/min |
| `Result` | Synthetic class label | Normal or Abnormal |

Example dataset:

[GeoAqua_Synthetic_IoT_Data.csv](GeoAqua_Synthetic_IoT_Data.csv)

The notebook generates the dataset with random seed 42 and exports the CSV.

These records are not field measurements. They contain no station coordinates or measurement timestamps.

The workflow combines satellite summaries and simulated sensor classification through decision rules. It does not match individual sensor records with satellite pixels or acquisition times.

## 5. Technical Approach

The notebook executes the following workflow:

1. Retrieve satellite scene metadata.
2. Select the first scene.
3. Display scene information and a map.
4. Download the surface reflectance HDF5 file.
5. Apply image quality filtering.
6. Select bands nearest the required wavelengths.
7. Calculate satellite indices.
8. Display index maps and spectral plots.
9. Generate 500 simulated IoT records.
10. Split records into training and test sets.
11. Train a Random Forest classifier.
12. Evaluate the classifier.
13. Classify the example sensor reading.
14. Display interactive sensor controls.
15. Apply combined satellite and sensor alert rules.
16. Display the dashboard and export files.

### Satellite Indices

`Rλ` represents reflectance at the band nearest wavelength λ in nanometres.

| Indicator | Formula | Purpose |
| --- | --- | --- |
| NDWI | `(R560 - R860) / (R560 + R860)` | Candidate water detection |
| NDCI | `(R708 - R665) / (R708 + R665)` | Relative chlorophyll proxy |
| Turbidity proxy | `(R665 - R560) / (R665 + R560)` | Relative optical turbidity indicator |
| NDVI | `(R800 - R665) / (R800 + R665)` | Supporting vegetation analysis |

The implementation adds a small value to denominators to reduce division errors.

### Random Forest Model

- Inputs: pH, turbidity, temperature, TDS, and flow
- Target: Normal or Abnormal
- Training records: 400
- Test records: 100
- Split: Stratified 80/20
- Number of trees: 100
- Maximum depth: 5
- Random state: 42
- Evaluation: Accuracy, precision, recall, F1 score, and confusion matrix

### Combined Alert Rules

The notebook identifies candidate water pixels using `NDWI > 0.1`.

Within these pixels, it calculates the 95th percentile of NDCI and the turbidity proxy.

An inspection alert appears when at least one condition applies:

- The example sensor reading is classified as Abnormal.
- The NDCI summary exceeds 0.20.
- The turbidity proxy summary exceeds 0.10.

These thresholds are demonstration parameters. They are not regulatory or water safety limits.

The current implementation uses an inspection alert or a normal result. A three-level framework would require additional implementation and validation.

## 6. Installation

### Google Colab

The observed Google Colab session uses Python 3.13.16. No GPU is required.

Internet access is required for installation and satellite downloads.

1. Open https://colab.research.google.com.
2. Select File → Open notebook → GitHub.
3. Enter https://github.com/noufmansoor/GeoAqua-Sentinel.
4. Open `GeoAqua_Sentinel_PoC_final.ipynb`.
5. Run the first installation cell.
6. Before the AI section, add and run this code cell:

   ```python
   %pip install pandas scikit-learn
   ```

7. Restart the session if prompted.
8. Run the notebook from the beginning.

The notebook installation commands use unpinned packages. Installation using the repository’s pinned `requirements.txt` still requires verification in a fresh environment.

### Local Jupyter

The following setup targets Python 3.12. This local setup has not yet been verified in a clean environment.

Clone the repository and create an environment:

```bash
git clone https://github.com/noufmansoor/GeoAqua-Sentinel.git
cd GeoAqua-Sentinel
python3.12 -m venv .venv
```

Activate on macOS or Linux:

```bash
source .venv/bin/activate
```

Activate on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies and start JupyterLab:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install jupyterlab
python -m jupyterlab
```

Open `GeoAqua_Sentinel_PoC_final.ipynb`.

Skip the notebook’s first installation cell when using the local pinned environment.

## 7. How to Run

Main notebook:

[GeoAqua_Sentinel_PoC_final.ipynb](GeoAqua_Sentinel_PoC_final.ipynb)

1. Complete installation.
2. Restart the notebook kernel or Colab session.
3. Run the analysis cells from first to last.
4. Wait for satellite data downloading and processing.
5. Check the model evaluation, combined alert, dashboard, and exported files.

The supplied example requires no scene or model changes. It selects the first scene in `item_ids`.

For an interactive sensor test, change pH, turbidity, temperature, TDS, and flow in the controls, then press Check Water.

Keep the demonstration thresholds unchanged when reproducing the example.

No API key is requested by the current notebook. Internet access is required for satellite metadata, downloads, installation, and map services.

### Runtime

The team measured approximately 2 minutes for a full notebook run in Google Colab. Runtime varies with download speed, cached data, dependency installation, and the computing environment.

### Final Outputs

The notebook displays:

- Random Forest evaluation and confusion matrix
- Interactive sensor controls
- Satellite index maps and summaries
- Example AI classification
- Combined inspection alert
- Combined monitoring dashboard

It exports:

- `GeoAqua_Synthetic_IoT_Data.csv`
- `GeoAqua_Sentinel_Dashboard.png`

Keep paths relative to the working directory.

## 8. Example Input and Output

### Example Input Data

[GeoAqua_Synthetic_IoT_Data.csv](GeoAqua_Synthetic_IoT_Data.csv) contains 500 simulated sensor records.

Satellite input is retrieved through the download code in the main notebook using the scene and asset selection documented in Section 4.

### Example Sensor Reading

| Parameter | Value |
| --- | --- |
| pH | 6.7 |
| Turbidity | 25 NTU |
| Temperature | 35 °C |
| TDS | 48,000 mg/L |
| Flow | 20 L/min |

### Expected Classification

```text
Accuracy: 1.0
AI Result: Abnormal
ALERT: Dangerous water condition detected!
```

The alert wording comes from the demonstration code. It does not establish a health or regulatory assessment.

### Combined Result

The saved demonstration reports approximately:

```text
Location: Tarif coastal area
Satellite NDCI: 0.009
Satellite Turbidity: -0.169
IoT and AI Result: Abnormal
FINAL ALERT: Water inspection is required.
```

Both satellite summaries are below their demonstration thresholds. The abnormal simulated sensor classification triggers the inspection alert.

### Dashboard

![GeoAqua Sentinel combined monitoring dashboard](GeoAqua_Sentinel_Dashboard.png)

Figure 1. Satellite indicator maps and the abnormal simulated sensor example.

### Confusion Matrix

![Random Forest confusion matrix](confusion%20matrix.png)

Figure 2. Classification results for 100 synthetic test records.

The confusion matrix appears during evaluation. The current notebook does not explicitly export its image.

## 9. Results and Limitations

### Classification Results

The saved notebook reports:

| Metric | Result |
| --- | --- |
| Accuracy | 100% |
| Precision, Normal | 100% |
| Precision, Abnormal | 100% |
| Recall, Normal | 100% |
| Recall, Abnormal | 100% |
| F1 score, Normal | 1.00 |
| F1 score, Abnormal | 1.00 |
| Correct Normal predictions | 50 of 50 |
| Correct Abnormal predictions | 50 of 50 |

The model trains on 400 synthetic records and evaluates on 100 held-out synthetic records.

The dataset generation and classification sections were reproduced during repository review. The generated records matched the uploaded CSV.

The dashboard and file export outputs were also observed in the team’s Colab run.

Installation from the pinned dependency file still requires a clean-environment test.

### Interpretation

Normal and Abnormal records use clearly separated generation ranges for several variables. This simplifies classification and explains the perfect test score.

The results demonstrate the synthetic classification workflow. They do not establish predictive accuracy under real coastal conditions.

Satellite indicators have not been validated against matching field samples or confirmed pollution events.

### Limitations

- All IoT records are synthetic.
- No physical sensor station has been tested.
- The demonstration analyzes one satellite acquisition.
- Sensor records lack coordinates and timestamps.
- Satellite and sensor observations are not spatially or temporally matched.
- Satellite indices are relative proxies rather than calibrated concentrations.
- Clouds, missing data, and invalid pixels reduce usable coverage.
- Demonstration thresholds require field calibration.
- The workflow does not certify safe water or identify the cause of an abnormal reading.
- The model does not confirm leakage.
- External downloads and services affect reproducibility.
- Runtime varies with downloads and the computing environment.
- The pinned installation has not yet been verified in a clean environment.

### Next Steps

- Build or obtain real sensor measurements.
- Record sensor coordinates and timestamps.
- Match measurements with satellite acquisition times.
- Calibrate satellite indicators against field samples.
- Evaluate independent real sensor records.
- Measure false alerts and missed abnormal events.
- Test multiple satellite dates.
- Develop dedicated leakage detection logic.
- Improve dashboard explanations and recommended actions.

## 10. Team, Licence and Attribution

Country representation: United Arab Emirates.

| Team member | Contribution |
| --- | --- |
| Alreem Ahmed Alkatheeri | Earth Observation and satellite data analysis |
| Nouf Mansoor Alblooshi | PoC notebook development, Google Colab implementation, and technical integration |
| Mouza Abdullah Almansoori | Study area analysis, documentation, and Tarif coastal area context |
| Marya Mohammed Alhammadi | Synthetic IoT dataset development and water monitoring variables |

### Satellite Attribution

The source notebook states:

CC BY 4.0, © Planet Labs PBC.

Retain the provider attribution when sharing derived figures and follow the applicable dataset terms.

### Source Notebook Attribution

The satellite workflow adapts educational material credited to Dr. Vincent Markiet, Space42, for the Arab Youth Space Hackathon 813 Challenge.

GeoAqua Sentinel adds synthetic IoT records, Random Forest classification, interactive sensor controls, combined alert rules, and a dashboard.

### Project Licence

No separate project licence file is currently included.

The satellite data licence does not automatically cover the team’s code or synthetic dataset. No additional software or dataset licence is claimed.

## Additional Repository Information

### Files

| File | Purpose |
| --- | --- |
| `README.md` | Documentation |
| `GeoAqua_Sentinel_PoC_final.ipynb` | Main analysis notebook |
| `requirements.txt` | Pinned dependency list |
| `GeoAqua_Synthetic_IoT_Data.csv` | Synthetic example data |
| `GeoAqua_Sentinel_Dashboard.png` | Example dashboard |
| `confusion matrix.png` | Example classification figure |
| `System_Architecture.png` | Proposed architecture |

### System Architecture

The proposed architecture connects satellite observations and sensor measurements to analysis, alerts, and a dashboard.

The current PoC implements satellite analysis and simulated sensor classification within a notebook.

![GeoAqua Sentinel system architecture](System_Architecture.png)
