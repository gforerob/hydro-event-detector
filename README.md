## Acknowledgments
This software allows users to Detect hydrological events identifying peaks and calculating volumes.

This package was developed by Gonzalo Forero Buitrago under the supervision of Dr. Humberto Vergara Arrieta at the University of Iowa, as part of the Advanced Hydrology and Warning Applications research at IIHR – Hydroscience & Engineering.

Special thanks to Dr. Humberto Vergara Arrieta for his guidance and support during the development of this software, and to Ruben Molina for generously sharing his knowledge.
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15650891.svg)](https://doi.org/10.5281/zenodo.15650891)

## Installation

    pip install hydro-event-detector

The HydroEventDetector works with these three main functions.
The first function of the HydroEventDetector package was named baseflow_lyne_hollick: The function separates baseflow and quick flow from streamflow time series using the Lyne-Hollick method. The Lyne-Hollick method is Recursive Digital Filter used for signal processing and is still used as a method for baseflow separation and characterize the difference between basins (Ladson, 2013). According to McMahon in 1990, the Lyne-Hollick digital filter was developed in Australia in a study using 180 basins having a good result for baseflow and quick flow separation from a streamflow time series. This method is extensively used for baseflow separation as at the R packages “lfstat” (Tobias Gauster, 2022) and “hydroEvents”  (Conrad Wasko, 2025).
The Lyne-Hollick method uses a parameter known as alpha value, that provide a height of the antecedent value to the present streamflow value. Several studies were developed studying the best alpha value for the baseflow and quick flow separation, such as McMahon 1990 suggest that the most acceptable base flow separation was in the range 0.9-0.95 and also Ladson 2013, that recommend using an alpha value of 0.98. For this specific study, after using different values for 30 USGS stations smaller than 1000 square kilometers and data every 15 minutes the alpha value of 0.987, showed the best results.
Ladson 2013 and McMahon in 1990 applied the Lyne and Hollick filter in two directions (forward and backward) and the basis can be founded in the signal processing theories. According to Smith 1997, a phase distortion is given by the direction where the recursive filter is applied. To cancel the phase distortion and preserve the original pulse shape, the filter must be applied in two directions (forward and backward), this original pulse shape is known as zero phase response (Smith, 1997.In conclusion, the recursive filter is applied in both forward and backward directions to preserve the fluctuations in the streamflow time series.

The Lyne and Hollick filter equations are:

q_f (i)=∝q_f (i-1)+((1+∝))/2  [q (i)-q(i-1)]  for q_((f)  ) (i)>0
   
q_b (i)=q(i)-q_f (i)

〖Where q〗_f (i)  is the quick flow response at the i instant
q(i)  is the streamflow value at i instant
q_b (i)  is the baseflow value at i instant
∝ is the alpha value

The second function of the of the HydroEventDetector package is detect_events: This function identify the crossing points between the baseflow and streamflow time series, and classify each crossing point into start and end. Within this start and end date ranges, the maximum value is founded as the peak value of the event.
The third function of the HydroEventDetector package, is filter_events: To filter the significant events a threshold is defined for the user and the threshold is a percentile of the "peak quick flow". The "peak quick flow" is the streamflow minus the baseflow value at the peak position. With the "peak quick flow" time series the user define a percentile. After applying this method to over 4,000 basins across the USA, the optimal percentile varies from 50 to 99%. For noisy streamflow signals, 99% is typically used, while for less noisy time series, percentiles between 50% and 99% can be appropriate. Although the method requires further experimentation, it performs well and the selected percentile can also serve as an indicator of the signal's variability.The best way to use it is to run the method on multiple basins using a single fixed percentile, count the number of detected events for each basin and then adjust the variable percentiles based on the results. This allows to ensure to obtaining statistically representative peak samples tailored to the specific study requirements.

To explore the package, a web platform was developed using 4,087 USGS stations. Users can apply the HydroEventDetector package, review the results through interactive plots, and download them as CSV files. The platform also allows users to adjust the date range and the alpha value used in the event detection process. The main goal is to explore hydrological events across different locations and investigate the influence of the alpha parameter and also the percentage, in the future can be possible to include another baseflow separation methods. 

Here is the link to the web page:

https://numericalflashfloodalertsolutions.com/maps/

NFFAS (Numerical Flash Flood Alert Solutions) is an interactive web-based platform designed to detect, visualize, calculate the volume and enable the download of significant hydrological events, including volume, date and flow data at the start, peak and end of each event for 4087 USGS stations, covering the period from April 2019 to March 2025. Processing is performed on the web using the Python package HydroEventDetector, enabling fast, reproducible, and scalable analysis while facilitating user interaction.  

To use the tool, select a state from the "Select a State" field. The colored points displayed on the map represent a flashiness classification based on the number of detected peaks. Click on a colored point, then click the "Show Chart" button. Next, choose the desired percentile and date range, and click "Refresh Plot." To download the results, click the "Download CSV" button.

References:

Conrad Wasko, D. G. (2021). Understanding event runoff coefficient variability across Australia using the hydroEvents R package. Hydrological Processes, 36(4). https://doi.org/10.1002/hyp.14563 
Conrad Wasko, D. G. (2025). Extract Event Statistics in Hydrologic Time Series. In https://github.com/conradwasko/hydroEvents
Ladson, A. R., Brown, R., Neal, B. & Nathan, R. (2013). A standard approach to baseflow separation using the Lyne and Hollick filter. Australian Journal of Water Resources, 17(1), 25-34. 
McMahon, R. J. N. a. T. A. (1990). Evaluation of Automated Techniques for Base Flow and Recessio Analyses. Water Resources Research, 26(7), 1465. https://doi.org/10.1029/WR026i007p01465  
Smith, S. W. (1997). The Scientist and Engineer's Guide to Digital Signal Processing. 
Tobias Gauster, G. L., Daniel Koffler. (2022). Calculation of Low Flow Statistics for Daily Stream Flow Data. In 0.9.12 (Version 0.9.12) https://cran.r-project.org/web/packages/lfstat/lfstat.pdf


# Hydro-Event-Detector – Example of Use

## ✅ 1. Install the Package

```bash
pip install Hydro-Event-Detector
```

> If you encounter installation errors, they are likely related to your environment. Please refer to the **[Library Conflict Troubleshooting](#library-conflict-troubleshooting)** section at the end of this file.

---

## 📚 2. Import Required Libraries

```python
from pathlib import Path
import pandas as pd
import hydro_event_detector as hed
from hydro_event_detector import HydroEventDetector
```

---

## 📂 3. Preprocess: Provide to the package the datetimes_naive and streamflow arrays required for processing (To ensure the method works correctly, make sure there are no NaN values within the time series. Fill in missing data or use only sections of the time series that are complete.)

### 🔹 Preprocess Example with Parquet Files

```python
HPC_HOME = Path().home() / "hpchome"
INPUT_FOLDER = HPC_HOME / "usgs_qa_rev"
STATIONS_LIST = HPC_HOME / "2025_April25_QA_Rev" / "codes_for_review" / "peak-hydro-event" / "src" / "usgs_june_pairing.csv"

df_codes = pd.read_csv(STATIONS_LIST, dtype={"USGS": str})
usgs_id = df_codes["USGS"].iloc[0]

file_path = INPUT_FOLDER / f"{usgs_id}.parquet"
df = pd.read_parquet(file_path)
df["datetime"] = pd.to_datetime(df["datetime"])
df = df.set_index("datetime").sort_index()
df = df[~df.index.duplicated(keep="first")]

# Separate valid values
complete = df["Discharge_cfs"]
valid = complete[complete.notna()]

# Convert to numpy format
datetimes_naive = pd.to_datetime(valid.index).tz_localize(None)
streamflow = pd.to_numeric(valid.values, errors='coerce')
```

---

### 🔹 Preprocess Example with NumPy Files

```python
import numpy as np

DATES_5MIN_PATH = Path("/home/gforerobuitrago/.../clean_dates.npy")
STATIONS_LIST = Path("/home/gforerobuitrago/.../usgs_june_pairing_timezone3.csv")
FLOW_FOLDER = Path("/home/gforerobuitrago/.../crest_npy_1d_final_por_estacion")

df_codes = pd.read_csv(STATIONS_LIST, dtype={"USGS": str})
usgs_id = df_codes["USGS"].iloc[0]

date_range_all = np.load(DATES_5MIN_PATH)
streamflow_all = np.load(FLOW_FOLDER / f"{usgs_id}.npy")

valid_mask = streamflow_all > 0
datetimes = pd.to_datetime(date_range_all[valid_mask]).tz_localize(None)
streamflow = streamflow_all[valid_mask]
```

---

## ⚙️ 4. Run the Package

```python
hed = HydroEventDetector(datetimes_naive, streamflow)

# Extract baseflow
hed.baseflow_lyne_hollick()

# Detect and filter events
hed.detect_events()
hed.create_events_dataframe()
full_df = hed.dataframe

hed.filter_events(90)  # 90 is a percentile threshold that you must change 
hed.create_events_dataframe()
filtered_df = hed.dataframe
```

---

## 📈 5. Plot Detected Events (Interactive Plot with Plotly)

```python
hed.plot_events("2019", "2021") #change the dates to plot
```

> If you face issues with `plotly`, use the HTML fallback method below.

---

## 🛠 6. HTML Fallback for Plotly (Optional)

```python
from types import MethodType

def patched_plot_events(self, start: str, end: str) -> None:
    import numpy as np
    import pandas as pd
    import plotly.graph_objects as go

    dates = self.date_range
    streamflow = self.streamflow
    baseflow = self.baseflow
    events_df = self.dataframe

    start_np = np.datetime64(start)
    end_np = np.datetime64(end)

    sf_series = pd.Series(streamflow, index=pd.to_datetime(dates))
    bf_series = pd.Series(baseflow, index=pd.to_datetime(dates))

    sf_slice = sf_series[start_np:end_np]
    bf_slice = bf_series[start_np:end_np]

    events_slice = events_df[
        (events_df["date_end"] >= start_np) &
        (events_df["date_start"] <= end_np)
    ]

    fig = go.Figure()
    fig.add_trace(go.Scatter(x=sf_slice.index, y=sf_slice.values, mode="lines", name="Streamflow"))
    fig.add_trace(go.Scatter(x=bf_slice.index, y=bf_slice.values, mode="lines", name="Baseflow", line=dict(dash="dot")))
    fig.add_trace(go.Scatter(x=events_slice["date_start"], y=events_slice["flow_start"], mode="markers", name="Start"))
    fig.add_trace(go.Scatter(x=events_slice["date_end"], y=events_slice["flow_end"], mode="markers", name="End"))
    fig.add_trace(go.Scatter(x=events_slice["date_peak"], y=events_slice["flow_peak"], mode="markers", name="Peak Flow"))
    fig.add_trace(go.Scatter(x=events_slice["date_peak"], y=events_slice["baseflow_peak"], mode="markers", name="Peak Baseflow"))

    fig.update_layout(title=f"Hydrologic Events From {start} To {end}", xaxis_title="Date", yaxis_title="Flow (cfs)")
    fig.write_html(f"plot_events_{start}_{end}.html")
    print(f"Plot saved as plot_events_{start}_{end}.html")

# Apply monkeypatch
hed.plot_events = MethodType(patched_plot_events, hed)

# Run it
hed.plot_events("2019", "2021")
```

---

## ⚠️ Library Conflict Troubleshooting

If you encounter issues installing or using the package, try the following steps:

```bash
# Step 1: Uninstall conflicting libraries in this example are pygeohydro bqplot nldi-xstool, this information is in the error
pip uninstall pygeohydro bqplot nldi-xstool -y

# Step 2: Upgrade pip
pip install --upgrade pip

# Step 3: Reinstall the package
pip install Hydro-Event-Detector

# Step 4: If NumPy causes issues, uninstall and install a compatible version
pip uninstall numpy
pip install numpy==1.26.4
```

Then verify:

```python
import numpy as np
print(np.__version__)
import hydro_event_detector as hed
```

---

## ✅ Done!

You’ve successfully run the Hydro-Event-Detector package.  
You can now analyze peak events, baseflow, and hydrologic volumes using the output dataframe.











