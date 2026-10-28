# Solar Irradiance Forecasting in Northern Vietnam

This project studies **Global Horizontal Irradiance (GHI) forecasting** in Hanoi, Son La, and Bac Giang, with an emphasis on model robustness when weather inputs are imperfect and on communicating forecast uncertainty through prediction intervals.

The project compares five forecasting approaches and proposes a two-phase framework:

1. **Robustness evaluation:** simulate weather-forecast errors with Monte Carlo noise injection and measure how forecast quality changes.
2. **Probabilistic calibration:** use Conformal Prediction to construct prediction intervals that account for model residuals and input uncertainty.

The target is the **clearness index** (\(K_t\)); predicted values are converted back to GHI for reporting in W/m². This is an irradiance forecasting project, not a solar power generation forecasting system.

## Study areas and data

The repository contains hourly CSV datasets for Hanoi, Son La, and Bac Giang, covering 2020–2025. Each file contains 52,601 rows and 18 columns, including timestamps, weather measurements, irradiance variables, and solar geometry variables.

The notebooks filter nighttime observations using the solar zenith angle and use 11 predictors:

- Cyclic time features: hour and month encoded as sine/cosine pairs
- Weather features: air temperature (`T2M`), relative humidity (`RH2M`), precipitation (`PRECTOTCORR`), and wind speed (`WS10M`)
- Solar features: solar zenith angle (`SZA`), extraterrestrial horizontal irradiance (`H0`), and clear-sky GHI (`CLRSKY_SFC_SW_DWN`)

Features that could leak information about the target or introduce redundancy are excluded, including diffuse irradiance, dew-point temperature, and wind direction. Standardization is fitted on the training data. The chronological split described in the report is:

| Split | Period |
|---|---|
| Training | January 2020 – February 2025 |
| Validation | March 2025 – August 2025 |
| Test | September 2025 – December 2025 |

## Models and methodology

The notebooks compare:

- Ridge Regression
- Random Forest
- XGBoost
- Multilayer Perceptron (MLP)
- Long Short-Term Memory (LSTM)

Hyperparameters are selected using the validation period. The strongest baseline for each location is then used for the subsequent uncertainty experiments:

| Location | Selected baseline |
|---|---|
| Hanoi | MLP |
| Son La | LSTM |
| Bac Giang | XGBoost |

For robustness testing, Gaussian noise is injected into selected weather inputs at low, medium, and high levels. The report describes Monte Carlo trials to quantify the impact on forecasts. For interval calibration, Conformal Prediction uses validation residuals; locally weighted calibration uses similar times of day. The calibrated intervals are widened to account for input uncertainty estimated during the robustness phase.

## Reported results

The report presents the following Prediction Interval Coverage Probability (PICP) after calibration:

| Location | Clean | Low noise | Medium noise | High noise |
|---|---:|---:|---:|---:|
| Hanoi | 85.0% | 89.6% | 92.0% | 94.1% |
| Son La | 85.1% | 88.7% | 90.3% | 92.4% |
| Bac Giang | 84.9% | 94.8% | 95.8% | 96.4% |

PICP is the proportion of observed values that fall inside the predicted intervals. Higher coverage generally comes with wider intervals, so coverage should be interpreted together with interval width and the intended use case. These are experimental results reported by the project and do not establish production performance.

## Repository structure

```text
.
├── Code/
│   └── EDA.ipynb                  # Exploratory data analysis
├── data/
│   ├── Feature Description.txt    # Feature definitions and units
│   ├── Power_solar_Bacgiang.csv
│   ├── Power_solar_Hanoi.csv
│   └── Power_solar_Sonla.csv
├── model/
│   ├── BacGiang_v2.ipynb          # Bac Giang experiments
│   ├── HaNoi.ipynb                # Hanoi experiments
│   └── Sonla_v2.ipynb             # Son La experiments
├── Reference/                     # Related research papers
├── REPORT_Group05.pdf             # Project report
├── SLIDE_Group05.pdf              # Presentation slides
└── README.md
```

## Getting started

### Requirements

Python 3 and Jupyter Notebook or JupyterLab are required. The notebooks use Python data-science and machine-learning packages, including:

```text
numpy
pandas
matplotlib
seaborn
scikit-learn
xgboost
tensorflow
shap
```

Install the packages in an environment compatible with your system:

```bash
python -m pip install numpy pandas matplotlib seaborn scikit-learn xgboost tensorflow shap jupyter
```

### Run the notebooks

1. Clone or download this repository and install the requirements.
2. Open `Code/EDA.ipynb` for exploratory analysis, or open a location-specific notebook under `model/`.
3. Update the CSV paths in the notebooks to point to the files under this repository's `data/` directory. The checked-in notebooks currently contain absolute paths from the original development environment.
4. Run the notebook cells in order.

For example, replace a machine-specific path with a path relative to the repository:

```python
from pathlib import Path
import pandas as pd

data_path = Path("data") / "Power_solar_Hanoi.csv"
df = pd.read_csv(data_path)
```

## Project files

- [Project report](REPORT_Group05.pdf) describes the problem, experimental design, and findings.
- [Presentation slides](SLIDE_Group05.pdf) summarize the project.
- `Reference/` contains research papers consulted during the work.

## Limitations

- Weather-input uncertainty is studied through simulated noise; the experiments do not establish performance using operational weather forecasts.
- The results are evaluated on the dataset and chronological split described in the report. Performance may differ for other periods, locations, or data sources.
- Notebook paths may need to be updated, and package versions are not pinned in a dependency lockfile.

## Acknowledgements

This is a group academic project. Please consult the project report for the full author list and institutional attribution.
