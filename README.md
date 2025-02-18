# haley_et_al_2024
> Analysis package for Haley et al. (2024)

These codes were written and used to analyze intracellular waveforms and extracellular nerve recordings from bursting neurons. Custom violin and stacked bar graph plotting scripts are included. This package is comprehensive and includes all codes used for this paper. Some of these codes are only useful for this particular set of experiments and will need to be adapted for other uses.


## Software

I used the following software for analysis and figure production:
- pClamp 10.5
- Spike2 v6.04
- MATLAB 2018a
- R v3.5.0
- RStudio 1.1.456
- Adobe Illustrator CC 2017


## File List

<b>Read `.abf` files in MATLAB</b>
- `abfload.m` : reads a `.abf` file (produced in pClamp) and produces a basic MATLAB structure
- `Loadabf.m` : wrapper for `abfload.m`; produces a more organized MATLAB struture for reading `.abf` files

<b>Analyze extracellular bursts in Spike2 and MATLAB</b>
- `batchImp.s2s` : given `.abf` files of type - ABF1(Integer) - which can be designated by exporting files in Clampfit, converts files to `.smr`, which is readable by other Spike2 scripts
- `spikenburst.s2s` : given a folder of `.smr` files, the gui walks you through defining bursts and spikes on extracellular traces using thresholding of membrane potential; produces `.txt` files for further anlaysis of spikes and bursts
- `readSpikeOutput.m` : given the `burst.txt` files produced by `spikenburst.s2s`, parces the data and produces simple measures of activity (e.g. frequency, spikes per burst, duration, duty cycle, start times, end times, and file numbers) for each burst
- `extracellularAnalysis.m` : for a particular experiment, this code uses `readSpikeOutput.m` and `convertpH.m` to pool and separate data into separate conditions; this function assumes that every condition was recorded in a separate `.abf` file; saves analysis as a structure `data.mat`

<b>Analyze intracellular waveforms in MATLAB</b>
- `analyzeWaveform.m` : given the waveform, sampling frequency, and default parameters, analyzes the slow wave and spiking activity of intracellular neurons
- `intracellularAnalysis.m` : for a particular experiment, this code uses `analyzeWaveform.m` to pool data for all conditions; this function assumes that every conditon was recorded in a separate `.abf` file; analyzes only the last minute of data; saves analysis as a structure `data.mat`

<b>Plotting data in MATLAB</b>
- `violinPlots.m` : given a 2D data matrix (columns are different conditions), creates violin plots; assumes 13 pH conditions, but code is simple and can be adapted for other purposes
- `violinPlotsControl.m` : similar to `violinPlots.m`, but assumes a 2D data matrix with columns as different preparations
- `stackedBarPlots.m` : produces stacked bar plots for state analysis
- `rectanglePlots.m` : produces colored boxes with saturation giving measure of rhythmicity
- `comparisonBarPlots.m` : produces colored boxes with saturation giving measure of rhythmicity for individual preparations at acid, base, and control

<b>Pool all experiments and plot in MATLAB</b>
- `pHCumulative.m` : accumulates all of the information from the `data.mat` files for each extracellular experiment included in the paper; produces all of the extracellular figures, saves data used in figures as `.csv` for statistics
- `pHCumulativePTX.m` : accumulates all of the information from the `data.mat` files for each intracellular PTX experiment included in the paper; produces all of the intracellular figures, saves data used in figures as `.csv` for statistics

<b>Statistical analyses in R/RStudio and MATLAB</b>
- `statistics.R`: code to compute a One-Way Repeated Measures ANOVA with post-hoc Paired Samples T-Tests (Bonferonni-corrected) or a Two-Way Multivariate ANOVA with post-hoc Independent Samples T-Tests (Bonferonni-corrected); reads `.csv` files outputed from `pHCumulative.m` or `pHCumulativePTX.m` and saves statistical analyses as new `.csv` files
- `formatStatTables.m` : reformats the `.csv` files produced in R to be more legible for figures; requires some tweeking in Microsoft Excel before being saved as an Adobe PDF for further cosmetic changes in Adobe Illustrator; no substantive utility


## Links

- Paper: [https://elifesciences.org/articles/41877](https://elifesciences.org/reviewed-preprints/103191)
- Git Repository: https://github.com/jesshaley/haley_et_al_2024
- Related projects: https://github.com/shreklab

  
## Contact
  
If you are having substantial issues or have questions about the code, please contact jess.allison.haley at gmail.com.
