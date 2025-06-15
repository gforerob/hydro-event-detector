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

To explore the results, a web platform was developed using 4,087 USGS stations. Users can apply the HydroEventDetector package, review the results through interactive plots, and download them as CSV files. Additionally, the platform allows users to adjust the dates and the alpha value used in the event detection process.

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


