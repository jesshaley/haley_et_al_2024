# haley_et_al_2024
> Analysis package for Haley et al. (2024)

This code was written and used to analyze the behavior of *C. elegans* exploring environments with small, dilute patches of bacteria. This package is comprehensive and includes all code used for this paper. Some files are only useful for this particular set of experiments and require adaption for other uses.


## Software

I used the following software for analysis and figure production:
- [MATLAB 2024a](https://www.mathworks.com/products/matlab.html)
- [WormLab](https://www.mbfbioscience.com/products/wormlab/)
- Adobe Illustrator 2024
- Adobe Photoshop 2024


## File List

**Compile metadata for a set of experiments**
- `getForagingInfo.m` : compiles all of the metadata for a set of experiments outlined in an **infoFile** (`.xls`) and saves that info as a table in a `.mat` file. This metadata includes information about the experiment (e.g. time stamps, conditions, camera, temperature), the video (e.g. pixels, frame rate), the behavioral arena (e.g. diameter, mask, orientation), and the lawns (e.g. mask, size, spacing).
  - **Get file metadata**
    - `getExperimentInfo.m` : retrieves metadata from a `.xls` file
    - `getVideoInfo.m` : retrieves metadata from a `.avi` video file
  - **Get arena properties**
    - `getAcetateArena.m` : takes an image from a video containing an acetate arena, locates the arena, and calculates information about the video
    - `getAcetateOrientation.m` : takes an image from a video containing both an acetate arena and a reference dot, locates the reference dot, and calculates information about the orientation of the arena
  - **Get lawn properties**
    - `getInvisibleLawns.m` : takes the experimental video and the contrast video (if applicable) and finds/estiamtes the location of bacterial lawns in the experiment
      - `getLawnsTemplate.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given information about size and spacing of a single lawn or an isometric grid of lawns.
      - `getLawnsThreshold.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given a frame from the video (ideally without worms) and information about the expected position of the lawns using image processing and morphological operations. `getLawnsThreshold.m` expects that the image used is not necessarily from the experimental video itself and enables translation, rotation, and scaling of the lawn mask to match the experimental video.
      - `getLawnsFilter.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given a VideoReader object (ideally for a video without worms) and information about the expected position of the lawns using image processing and morphological operations. `getLawnsFilter.m` expects a video where a dark object (e.g. black cardstock) is passed between the lightsource and the experimental plate, creating a "darkfield" image of plate. The cardstock scatters the light creating increased contrast of the bacteral lawns, enabling automatic identification of the lawn edges. `getLawnsFilter.m` expects that the video used contains an arena that is not necessarily in the same position and orientation as the experimental video itself and enables translation, rotation, and scaling of the lawn mask to match the experimental video.
      - `getLawnsCircle.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given a VideoReader object (ideally for a video without worms) and information about the expected position of the lawns using image processing and morphological operations. `getLawnsCircle.m` expects a video where a dark object (e.g. black cardstock) is passed between the lightsource and the experimental plate, creating a "darkfield" image of plate. The cardstock scatters the light creating increased contrast of the bacteral lawns, enabling automatic identification of the lawn edges. `getLawnsCircle.m` uses [`imfindcircles`](https://www.mathworks.com/help/images/ref/imfindcircles.html?searchHighlight=imfindcircles&s_tid=srchtitle_support_results_1_imfindcircles) to  identify the putative lawns on every frame and creates a weighted average image. `getLawnsCircle.m` expects that the video used contains an arena that is not necessarily in the same position and orientation as the experimental video itself and enables translation, rotation, and scaling of the lawn mask to match the experimental video.
  - **Create grids** (could be used to analyze spatial properties of animal tracks)
    - `createHexagonalGrid.m` : creates a grid of isometric hexagons and corresponding triangles (6 per hexagon) given information about their size and spacing
    - `createDiamondGrid.m` : creates a grid of diamonds and corresponding triangles (2 per diamond) given information about their size and spacing
    - `transformLabeledImage.m` : takes in an image and its properties as a structure and performs given image transformations (i.e. registrations); performed on the grids (hexagon, diamond, triangle) to register them to arena



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

- Paper: https://elifesciences.org/articles/41877
- Git Repository: https://github.com/jesshaley/haley_et_al_2024
- Related projects: https://github.com/shreklab

  
## Contact
  
If you are having substantial issues or have questions about the code, please contact jess.allison.haley at gmail.com.
