# haley_et_al_2024
> Analysis package for [Haley et al. (2024)](https://elifesciences.org/articles/41877)

This code was written and used to analyze the behavior of *C. elegans* exploring environments with small, dilute patches of bacteria. This package is comprehensive and includes all code used for this paper. Some files are only useful for this particular set of experiments and require adaption for other uses.

# Table of Contents
1. [Software](#software)
2. [File list](#file-list)
   - [Compile experiment metadata](#foragingInfo)
     - [Get file metadata](#metadata)
     - [Get arena properties](#arena)
     - [Get lawn properties](#lawn)
     - [Image registration](#registration)
     - [Create grids](#grid)
   - [Analyze worm behavior](#behavior)
     - [Analyze worm movement and identify encounters](#analysis)
     - [Plot behavior](#plot)
   - [Classify worm behavior](#label)
   - [Estimate bacterial density](#OP50-GFP)
   - [Model the exploitation decision](#model)
   - [Statistical analysis](#statistics)
3. [Links](#links)
4. [Contact](#contact)

## Software <a name="software"></a>

I used the following software for analysis and figure production:
- [MATLAB 2024a](https://www.mathworks.com/products/matlab.html)
- [WormLab](https://www.mbfbioscience.com/products/wormlab/)
- Adobe Illustrator 2024
- Adobe Photoshop 2024


## File list <a name="file-list"></a>

### Compile experiment metadata <a name="foragingInfo"></a>
- `getForagingInfo.m` : compiles all of the metadata for a set of experiments outlined in an **infoFile** (`.xls`) and saves that info as a table in a `.mat` file. This metadata includes information about the experiment (e.g. time stamps, conditions, camera, temperature), the video (e.g. pixels, frame rate), the behavioral arena (e.g. diameter, mask, orientation), and the lawns (e.g. mask, size, spacing).

  - #### Get file metadata <a name="metadata"></a>

    - `getExperimentInfo.m` : retrieves metadata from a `.xls` file.
    - `getVideoInfo.m` : retrieves metadata from a `.avi` video file.

  - #### Get arena properties <a name="arena"></a>

    - `getAcetateArena.m` : takes an image from a video containing an acetate arena, locates the arena, and calculates information about the video.
    - `getAcetateOrientation.m` : takes an image from a video containing both an acetate arena and a reference dot, locates the reference dot, and calculates information about the orientation of the arena.

  - #### Get lawn properties <a name="lawn"></a>

    - `getInvisibleLawns.m` : takes the experimental video and the contrast video (if applicable) and finds/estimates the location of bacterial lawns in the experiment.
      - `getLawnsTemplate.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given information about size and spacing of a single lawn or an isometric grid of lawns.
      - `getLawnsThreshold.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given a frame from the video (ideally without worms) and information about the expected position of the lawns using image processing and morphological operations. `getLawnsThreshold.m` expects that the image used is not necessarily from the experimental video itself and enables translation, rotation, and scaling of the lawn mask to match the experimental video.
      - `getLawnsFilter.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given a VideoReader object (ideally for a video without worms) and information about the expected position of the lawns using image processing and morphological operations. `getLawnsFilter.m` expects a video where a dark object (e.g. black cardstock) is passed between the lightsource and the experimental plate, creating a "darkfield" image of plate. The cardstock scatters the light creating increased contrast of the bacteral lawns, enabling automatic identification of the lawn edges. `getLawnsFilter.m` expects that the video used contains an arena that is not necessarily in the same position and orientation as the experimental video itself and enables translation, rotation, and scaling of the lawn mask to match the experimental video.
      - `getLawnsCircle.m` : creates an estimate of the locations of bacterial lawns in the behavioral arena given a VideoReader object (ideally for a video without worms) and information about the expected position of the lawns using image processing and morphological operations. `getLawnsCircle.m` expects a video where a dark object (e.g. black cardstock) is passed between the lightsource and the experimental plate, creating a "darkfield" image of plate. The cardstock scatters the light creating increased contrast of the bacteral lawns, enabling automatic identification of the lawn edges. `getLawnsCircle.m` uses [`imfindcircles`](https://www.mathworks.com/help/images/ref/imfindcircles.html) to identify the putative lawns on every frame and creates a weighted average image. `getLawnsCircle.m` expects that the video used contains an arena that is not necessarily in the same position and orientation as the experimental video itself and enables translation, rotation, and scaling of the lawn mask to match the experimental video.
    - `getManualLawns.m` : updates lawn locations with any manual corrections that were made in Adobe Photoshop (when needed).

  - #### Image registration <a name="registration"></a>

    - `registerImage.m` : takes in two images, detects SURF features (using [`detectSURFFeatures`](https://www.mathworks.com/help/vision/ref/detectsurffeatures.html)) of each, estimates a 2D rigid transformation (using [`estgeotform2d`](https://www.mathworks.com/help/vision/ref/estgeotform2d.html)), and performs the transformation (using [`imwarp`](https://www.mathworks.com/help/images/ref/imwarp.html)). 
    - `transformLabeledImage.m` : takes in an image and its properties as a structure and performs given image transformations (i.e. registrations); performed on the grids (hexagon, diamond, triangle) to register them to arena. This function use simple geometric tranformations (e.g., [`imrotate`](https://www.mathworks.com/help/images/ref/imrotate.html)) and is intended for use with images that have been labeled using [`bwlabel`](https://www.mathworks.com/help/images/ref/bwlabel.html). If using a video frame, use `registerImage.m` for more accurate image registration.

  - #### Create grids <a name="grid"></a>

    - `createHexagonalGrid.m` : creates a grid of isometric hexagons and corresponding triangles (6 per hexagon) given information about their size and spacing.
    - `createDiamondGrid.m` : creates a grid of diamonds and corresponding triangles (2 per diamond) given information about their size and spacing.

### Analyze worm behavior <a name="behavior"></a>
- `analyzeForaging.m` : uses the body segment location of animal combined with the location of bacterial patches/lawns to identify encounters and properties of those encounters.

   -  #### Analyze worm movement and identify encounters <a name="analysis"></a>
    
      - `analyzeWormLabTracks.m` : takes body segment data exported from WormLab and computes numerous metrics related to the worm's position, velocity, and turning behavior as well as it's location relative to the arena and lawns.
      - `offsetTime.m` : corrects the wormNum identifiers and time values for experiments where the recording spans multiple `.avi` files.
      - `defineEncounter.m` : uses high-resolution body segment locations of animals exploring environments with a single small patch of bacteria to define an encounter event for use when only the mid-point of the animal can be reliably tracked.
      - `analyzeEncounters.m` : uses the location of animal(s) and lawn(s) to identify encounters and properties of those encounters.
   
    - #### Plot behavior <a name="plot"></a>
      - `plotTracks.m` : writes an image to file showing the tracks of an animal overlaid onto an image of the lawns and arena. Tracks are colored based on a given metric (e.g., time, velocity, path angle).
      - `createVideo.m` : writes a video to file showing either the tracks of an animal or the original video downsampled with scale bar and time stamp showing.

*this following section(s) are in progress as of 2/19/2025*

### Classify worm behavior <a name="label"></a>
- `labelEncounters.m` : uses the body segment location of an animal combined with the location of bacterial patches/lawns to identify encounters and properties of those encounters.
   - `estimatePosterior.m` :

### Estimate bacterial density <a name="OP50-GFP"></a>
- `analyzeGFP.m` : gets all the meta data for a set of experiments, identifies brightfield images of acetate templates, identifies background images of empty plates, and extracts and analyzes the fluorescence intensity profile for each patch.
   - #### Get file metadata <a name="metadata2"></a>
      - `getPlateInfo.m` :
      - `getMetaDataCZI.m` :
   - #### Analyze fluorescence <a name="fluorescence"></a>
      - `getFlourescenceBackground.m` :
      - `analyzeLawnProfiles.m` :
- `plotForagingGFP.m` :

### Model the exploitation decision <a name="model"></a>
- `modelExploit.m` :

### Statistical analyses <a name="statistics"></a>
- `benjaminiHochberg.m` :
- `silvermansTest.m` :
- `writeSourceData.m` :

## Links <a name="links"></a>

- Paper: https://elifesciences.org/articles/41877
- Git Repository: https://github.com/jesshaley/haley_et_al_2024
- Related projects: https://github.com/shreklab

  
## Contact <a name="contact"></a>

If you are having substantial issues or have questions about the code, please contact jess.allison.haley at gmail.com.
