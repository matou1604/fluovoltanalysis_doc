
# 2. Signal analysis

The second part of the analysis is done using a **Python script**. This script takes as input the CSV file generated by the ImageJ plugin and outputs 3 different CSVs and 2 PNGs, concluding the analysis and storing all results. Following this step, normalisation and statistical analysis are still required.


## User guide 

### a. Installation

The script is available for download [here](https://github.com/matou1604/fluovoltanalysis_py/blob/main/feature_extraction.py). 

Once downloaded, you can drag the `.py` file into a specific folder of your choice. You can rename the file if you want to. As code editor, it is recommended to download *Visual Studio Code* for a better experience when running the code. You can download it [here](https://code.visualstudio.com/).

The following packages are required to run the script:
- [Python](https://www.python.org/downloads/)
  - [Pandas](https://pandas.pydata.org/)
  - [Matplotlib](https://matplotlib.org/)
  - [Numpy](https://numpy.org/)
  - [Scipy](https://www.scipy.org/)
  - [Seaborn](https://seaborn.pydata.org/)

You will therefore need to install them. To do so, download python to your device (if not done already) and type the following commands one after the other in the terminal at the location of your file:

```bash
pip install pandas
pip install matplotlib
pip install numpy
pip install scipy
pip install seaborn
```
You can also create an environment with the required packages.

Once you're set up, open the script in VS Code and run it in an *interactive window* (see [next section](#b-running-the-script)).

<br>

### b. Running the script

To run, right-click on the script an select ***Run in interactive window* > *Run current file in Interactive window***. This will open a new pane on the right and print all outputs there. You might need to dowload *Jupyter Notebooks* directly in VS code to be able to do this.

Before running the script you might want to adjust the following constants:
- `ALL_FOLDERS`: set to `True` if you want to analyze all subfolders in the selected folder, or `False` if you only want to analyze one subfolder.
- `PEAK_THRESHOLD`: the threshold height for the peaks and troughs detection. The default value is 0.3, so the peaks can only be detected if they are in the highest 30% of the signal, and the troughs in the lowest 30%.
- `MIN_DISTANCE_DIVIDER`: the minimum distance between peaks is computed as the period in points divided by 3.5, by default. You can replace 3.5 by a smaller number to be more strict, but you might miss peaks if the signal is irregular.

The script will open a directory window for choosing the folder to analyse; and will then start the analysis. 

Here is a preview of the outputs for one signal:

```
1. Analyzing condition: Basal1

1/19. Processing file:  AK12_040225_D14_20x obj_fluovolt_Basal1_A6-1
```

<div style="padding: 5px;">
  <img style="text-align: center;" src="_images\D14_Basal2_F2f_raw.png">
</div>

<div style="padding: 5px;">
  <img style="text-align: center;" src="_images\D14_Basal2_F2f_apd.png">
</div>

<div style="padding: 5px; ">
  <img style="text-align: center; width:70%" src="_images\D14_Basal2_F2f_apdonebeat.png">
</div>

```
Is the analysis of this signal good? 
Press Enter if 'Yes', or type anything else if 'No'
```

If the analysis is good, press Enter. If not, type anything else and press Enter. The results of the badly analyzed signal will still be saved, but will be stored with the boolean *Good analysis* as False. The script will then move on to the next signal.

```{warning}
VS Code has an output number limit at 500 by default. Change this number in the settings to avoid missing any outputs when your're halfway through the analysis:
```
<img style="text-align: center; width:93%; margin-top: 0px;" src="_images/size_limit.png"><img style="text-align: center; width:70%;" src="_images/scroll.png">

<br>

### c. Results

The script will output 2 CSV files and 2 PNGs for each signal analyzed. One additional CSV file is generated per condition/subfolder. 

The CSV files are:

- `_signal.csv:` contains the interpolated raw signal data, and the interpolated, normalized, baseline subtracted and smoothed version after preprocessing. The interpolated time points for each mean intensity value is also included.

- `_apd.csv:` stores the computed Action Potential Duration (APD) data for each beat. See the explanation for the different calculated APD in the [next section](#python-code). Peaks, troughs, and pretroughs are also saved in this file, where each row corresponds to a single beat. X and Y values of all points are also stored in this file, to allow replottings.

- `_param_means.csv:` aggregates all extracted features and metrics of the analysis of one condition or subfolder. Here each row repesents one signal and its extracted metrics such as period, frequency, rise time, amplitude, and APD means. The *Good analysis* boolean values are also saved as a column here to keep track of the quality of the analysis. To understand how each parameter was calculated, refer to the [next section](#python-code).

The PNG files are:

- `_plot.png:` displays the processed signal with detected peaks, troughs, and APD points overlaid. This helps the user to verify the accuracy of the analysis.

- `_singleplot.png:` focuses on a single beat, the second one. this illustrates APD results in detail to highlight signal dynamics within an individual cycle.


<br>

## Python code

Once loaded, the data undergoes preprocessing, which includes interpolation, smoothing with a savgol filter, and baseline subtraction.

Next, the script detects peaks, troughs, and pretroughs in the signal. Peaks represent the highest points, troughs are the lowest points after a peak, and pretroughs are the lowest points before a peak. The alignment of these points is checked to ensure valid signal cycles. If any inconsistencies are found, the signal is skipped. 

At this point, the signal is normalized to the baseline, which is computes as the mean of the troughs.

The script then computes key metrics such as period and frequency, signal amplitude, rise time, interpeak intervals, and action potential duration (APD) at different percentages. The APD values measure how long it takes for the signal to repolarize after the peak, for studying signal behavior and potentially arrhythmias.

Here is a list of all parameters and how they are computed: 
- **Period (s):** the mean of the time between two consecutive peaks.
- **Frequency (Hz):** the number of peaks per minute, calculated as 1/period.
- **Interpeak std (s):** the standard deviation of the time between two consecutive peaks.
- **Action potential (s):** the mean of the time between pretrough and trough for each beat, should be the same as APD100off.
- **Rise time (s):** the mean of the time it takes for the signal to rise from the pretrough to the peak for each beat.
- **Max rise slope (a.u./s):** the mean of the maximum slope (speed) of the signal between the pretrough and the peak for each beat.
- **Amplitude (a.u.):** the mean of the difference between the peak and the trough for each beat.
- **Amplitude std (a.u.):** the standard deviation of the amplitudes of each beat.
- **Triangulation (apd50/apd90):** the mean of the ratio between the APD50 and the APD90 for each beat.
- **Good analysis:** a boolean value indicating whether the analysis is accurate. This value was entered by the user during the script running.
- **APDx mean (s):** the time it takes for the signal to repolarize of a certain percentage from the peak value. The APD is calculated for 10%, 20%, 30%, 40%, 50%, 60%, 70%, 80%, and 90% of the peak value. In this case the start point is at 1-X% of the rise. The non-averaged APD values can be found in the `_apd.csv` file, along with time-points and values for each beat.
- **APDxoff mean (s):** the time it takes for the signal to repolarize to a certain percentage of the peak value. The APD is calculated for 10%, 20%, 30%, 40%, 50%, 60%, 70%, 80%, and 90% of the peak value. In this case the start point is the peak. The non-averaged APD values can be found in the `_apd.csv` file, along with time-points and values for each beat.

The results are saved as CSV files, and plots are generated to visualize the main extracted features. The user is then prompted to confirm whether the analysis is accurate and the answer is stored in a boolean value. 

