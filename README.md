# hydro-event-detector
Detect hydrological events identifying peaks and calculating volumes

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15650891.svg)](https://doi.org/10.5281/zenodo.15650891)

## Installation

    pip install hydro-event-detector

The HydroEventDetector works with these three main functions.
The first function of the HydroEventDetector package was named baseflow_lyne_hollick, that separates baseflow and quick flow from streamflow time series using the Lyne-Hollick method. The Lyne-Hollick method is Recursive Digital Filter used for signal processing and is still used as a method for baseflow separation and characterize the difference between basins (Ladson, 2013). According to McMahon in 1990, the Lyne-Hollick digital filter was developed in Australia in a study using 180 basins having a good result for baseflow and quick flow separation from a streamflow time series. This method is extensively used for baseflow separation as at the R packages “lfstat” (Tobias Gauster, 2022) and “hydroEvents”  (Conrad Wasko, 2025).
The Lyne-Hollick method uses a parameter known as alpha value, that provide a height of the antecedent value to the present streamflow value. Several studies were developed studying the best alpha value for the baseflow and quick flow separation, such as McMahon 1990 suggest that the most acceptable base flow separation was in the range 0.9-0.95 and also Ladson 2013, that recommend using an alpha value of 0.98. For this specific study, after using different values for 30 USGS stations smaller than 1000 square kilometers and data every 15 minutes the alpha value of 0.987, showed the best results.
Ladson 2013 and McMahon in 1990 applied the Lyne and Hollick filter in two directions (forward and backward) and also at the HydroEventDetectorpackage. The reason can be founded in the signal processing theories. According to Smith 1997 the phase distortion is given by the direction when a filter is applied. To cancel the phase distortion and preserve the original pulse shape, the filter must be applied in two directions (forward and backward), this original pulse shape is known as zero phase response (Smith, 1997).
The Lyne and Hollick filter equations are:

q_f (i)=∝q_f (i-1)+((1+∝))/2  [q (i)-q(i-1)]  for q_((f)  ) (i)>0
   
q_b (i)=q(i)-q_f (i)

〖Where q〗_f (i)  is the quick flow response at the i instant
q(i)  is the streamflow value at i instant
q_b (i)  is the baseflow value at i instant
∝is the alpha value

The second function of the of the HydroEventDetector package, named as detect_events, identify the crossing points between the baseflow and streamflow time series and classify each crossing point into start and end. Within this start and end date ranges, the maximum value is founded as the peak value of the event.
The third function of the HydroEventDetector package, named as filter_events, filter the significant events. To filter the events, at the peak position the package calculate the peak quick flow that is the streamflow minus the baseflow value at the peak position. With the peak quick flow time series the 90th percentile is calculated and then only the significant events or the events which its peak quick flow is over the 90th percentile will be used as a significant event, this percentile can be changed depending on the results in the HydroEventDetector package .

To explore the results, a web platform was developed using 4,087 USGS stations. Users can apply the HydroEventDetector package, review the results through interactive plots, and download them as CSV files. The platform also allows users to adjust the date range and the alpha value used in the event detection process. The main goal is to explore hydrological events across different locations and investigate the influence of the alpha parameter and also is open the possibility of include another baseflow separation methods. This aspect requires further study and experimentation, and we invite collaborators to participate and contribute to this ongoing effort.

Here is the link to the web page:

https://numericalflashfloodalertsolutions.com/maps/

NFFAS (Numerical Flash Flood Alert Solutions) is an interactive web-based platform designed to detect, visualize, calculate the volume and enable the download of significant hydrological events, including volume, date and flow data at the start, peak and end of each event, for 4087 USGS stations, covering the period from April 2019 to March 2025. Processing is performed on the web using the Python package HydroEventDetector, enabling fast, reproducible, and scalable analysis while facilitating user interaction.  

To use the tool, select a state from the "Select a State" field. The colored points displayed on the map represent a flashiness classification based on the number of detected peaks. Click on a colored point, then click the "Show Chart" button. Next, choose the desired percentile and date range, and click "Refresh Plot." To download the results, click the "Download CSV" button.

References:

Conrad Wasko, D. G. (2021). Understanding event runoff coefficient variability across Australia using the hydroEvents R package. Hydrological Processes, 36(4). https://doi.org/10.1002/hyp.14563 
Conrad Wasko, D. G. (2025). Extract Event Statistics in Hydrologic Time Series. In https://github.com/conradwasko/hydroEvents
Ladson, A. R., Brown, R., Neal, B. & Nathan, R. (2013). A standard approach to baseflow separation using the Lyne and Hollick filter. Australian Journal of Water Resources, 17(1), 25-34. 
McMahon, R. J. N. a. T. A. (1990). Evaluation of Automated Techniques for Base Flow and Recessio Analyses. Water Resources Research, 26(7), 1465. https://doi.org/10.1029/WR026i007p01465  
Smith, S. W. (1997). The Scientist and Engineer's Guide to Digital Signal Processing. 
Tobias Gauster, G. L., Daniel Koffler. (2022). Calculation of Low Flow Statistics for Daily Stream Flow Data. In 0.9.12 (Version 0.9.12) https://cran.r-project.org/web/packages/lfstat/lfstat.pdf

# Example of use !!!!
1. Manage libraries conflict. Install the package and review if you got errors, if you got errors are related with your environment, read the chapter libraries conflict at the end of this file.

pip install Hydro-Event-Detector

2. Import libraries
   
from pathlib import Path

import pandas as pd

import hydro_event_detector as hed
from hydro_event_detector import HydroEventDetector

3. Read the files that you want to process, here I will provide an example with parquet and with numpy but can be other formats as well

# Read the paths
HPC_HOME = Path().home() / "hpchome" # example using a server, not neccesary it is just part of the path used 
INPUT_FOLDER = HPC_HOME / "usgs_qa_rev" # folder with the files that you want to process
STATIONS_LIST = HPC_HOME / "2025_April25_QA_Rev" / "codes_for_review" / "peak-hydro-event" / "src" / "usgs_june_pairing.csv" # list of USGS stations codes to process

# Read the data
df_codes = pd.read_csv(STATIONS_LIST, dtype={"USGS": str})
usgs_id = df_codes["USGS"].iloc[0]
file_path = INPUT_FOLDER / f"{usgs_id}.parquet"

df = pd.read_parquet(file_path)
df["datetime"] = pd.to_datetime(df["datetime"])
df = df.set_index("datetime").sort_index()
df = df[~df.index.duplicated(keep="first")]

# --- Keep nans and separate for calculation ---
complete = df["Discharge_cfs"]
valid = complete[complete.notna()]

# --- Valids to numpy ---
#datetimes = valid.index.to_numpy()
datetimes_naive = pd.to_datetime(valid.index).tz_localize(None)
streamflow = pd.to_numeric(valid.values, errors='coerce')

# Extract event peaks, volumes and baseflow

hed = HydroEventDetector(datetimes_naive, streamflow)
hed.baseflow_lyne_hollick()
#hed.baseflow
hed.detect_events()
hed.create_events_dataframe()
full_df = hed.dataframe
#full_df
hed.filter_events(90)
hed.create_events_dataframe()
filtered_df = hed.dataframe
filtered_df

# Interactive plot 

hed.plot_events("2019", "2021") # Run this and change the interested dates 

# If do you have plotly conflicts use html (Not necessary, use just if the previous plot does not work)

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
    print(f"plot saved as plot_events_{start}_{end}.html")

# Apply monkeypatch
hed.plot_events = MethodType(patched_plot_events, hed)

# Execute
hed.plot_events("2019", "2021") #change the interested dates

# You successfully run the package here. Explore the information related with peaks, volume, baseflow.

________________________________________________________________________________
# Libraries conflict chapter in the case you faced problems running the package

pip install Hydro-Event-Detector
# if you have an conflict with other libreries you can unstall them
#Example for unstall the libraries with conflict
pip uninstall pygeohydro bqplot nldi-xstool -y
#upgrade pip if its required
%pip install --upgrade pip
#try again
pip install Hydro-Event-Detector
# if your environment has problems with the new numpy versions unstall it and install a numpy version that works 
%pip uninstall numpy
# Example of install a version that workd for your environment
%pip install numpy == 1.26.4
#    import numpy as np
np.__version__
# import the library to review that works now
import hydro_event_detector as hed
# if you are successfull you can go directly to the import libraries or number 2.





