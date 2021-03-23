# Purpose

Save figures in an editable format for publication.


# premessa

`premessa` is an R package for pre-processing of flow and mass cytometry data, that includes panel editing/renaming for FCS files, bead-based normalization and debarcoding.

Copyright 2016. Parker Institute for Cancer Immunotherapy

**Please contact us with bugs/feature requests** 

**---> Make sure to have a backup copy of your data before you use the software! <---**


# Installation


## Install required R packages

You need to install the devtools package, available from CRAN, and the flowCore package from Bioconductor. The rest of the dependencies for premessa will be automatically installed

#### devtools

Open an R session, type the following command and select a CRAN mirror when prompted.

```
install.packages("devtools")
```

#### flowCore

Open an R session and type the following commands

```
source("http://bioconductor.org/biocLite.R")
biocLite("flowCore")
```

## Install premessa

Start an R session and type the following commands

```
library(devtools)
install_github("lshh125/premessa")
```

This will install the premessa R package together with all the required dependencies.

*Note*: the latest version of devtools seems to be occasionally having problems installing dependencies on Windows. If the installation of premessa fails due to a missing package, please install the offending packages manually, using the R *install.packages* function


# Usage

The software allows you to perform four operations:
- [Panel editing and renaming](#panel-editing-and-renaming)
- [FCS file concatenation](#fcs-file-concatenation)
- [Bead-based normalization](#bead-based-normalization)
- [De-barcoding](#de-barcoding)

For each operation there is a separate GUI, and an associated set of R functions that can be called wihtout using the GUI, if desired

## Panel editing and renaming

`premessa` includes a component for editing and renaming the panel of a set of FCS files. This is useful when you need to harmonize panels across a number of files, so that they can be prepared for downstream analysis (most analysis tools expect files that are part of the same analysis to have identical panels).

A few warnings. `premessa` is *opinionated* in the way it handles the information in the FCS panels, and is specifically tailored to handle the most common use case. The FCS file specification is problematic in a lot of ways and most instrument and analysis software packages do not use or interpret the information correctly anyways. There are 3 ways to refer to a channel in an FCS file, by number (e.g. the order in which they appear in the file), by name (e.g. *Dy161Di* ) and by description (e.g. *CD3*). `premessa` uses the name as unique channel identifier for two reasons:
- it is guaranteed to be unique in a valid FCS file
- it minimizes the risk of confusion when matching channels between multiple FCS files, as it corresponds to the intuitive notion of matching channels based on their identity instead of their ordering.

The consequence of this choice is that **the ordering of the channels is not preserved during the processing**. Also at present `premessa` only preseves the `name` and `description` parameter keywords (i.e. $PnN and $PnS). All other parameter keywords (e.g. $PnG, $PnL, $PnO etc.) are discarded. Most of these keywords are used incorrectly anyways, but please feel free to open an issue if this is impacting your workflow.

## Starting the GUI and selecting the working directory

You can start the panel editor GUI by typing the following commands in your R session

```
library(premessa)
paneleditor_GUI()
```
This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory containing the data you want to analyze, and select any file in that directory. The directory itself will then become the working directory for the software.

To stop the software simply hit the "ESC" key in your R session.

## Usage

Once you have selected the working directory, the software will extract the panel information from all the FCS files contained in the directory. This information is then displayed in a table, where each row corresponds to a different parameter name ($PnN keyword), indicated by the row names (leftmost column), and each column corresponds to a different file, indicated in the column header. Each cell represents the description string ($PnS keyword) of a specific parameter in a given file. If a parameter is missing from a file, the word *absent* is displayed in the corresponding cell, which will be colored orange (note that this means that *absent* cannot be a valid parameter name)

Whatever is written in the table when the *Process files* button is pressed, represents what the parameters will be renamed to. In other words the table represents the current state of the files, and you have to edit the individual cells as necessary to reflect the desired final state of the files. You can use the same shortcuts you use in Excel to facilitate the editing process (e.g. shift-click to select multiple rows or columns, ctrl-C and ctrl-V for copy and paste respectively, etc.). However be careful that pressing ctrl-Z (the conventional undo shortcut) will undo *all* you changes

The table begins with three special columns:
- *Remove*: if the box is checked the corresponding parameter is removed from all the files, and the row is grayed out
- *Parameter*: this column represent the parameter name ($PnN keyword). Initially it will be identical to the first column of row names, but this column is editable. You can edit this column if you want to change the parameter names in the output files.
- *Most common*: this column indicates what is the most common description value for that parameter, across all the files under analysis (i.e. the most common string across the row). Cells whose value differs from the value indicated in this column are displayed with a light pink background.

The table columns are sorted by the number of *problematic* columns, i.e. by the number of pink and orange cells in the column. The first three columns of the table are fixed and always visible when you scroll the table horizontally. Please note that the browser included with the current vesion of RStudio seems to have a problem where the column headers do not scroll correctly. If that is the case, open the application in a regular web browser, by click on the "open browser" button in the top right corner of the RStudio browser.

Two controls are located at the top of the table
- *Output folder name*: a text box where you can input the name of the output folder. If this folder does not exist, it will be created as a sub-folder of the current working directory.
- *Process files*: this button will start file processing. A file will be created in the output folder, with the same name as the original input file. **If a file of the same name exists already, it will be overwritten**. No change is made to the original files.




## FCS file concatenation

The `premessa` package contains a simple function for concatenating multiple FCS files together. This is useful in case the acquisition of a single samples has been split across multiple files. The function is called `concatenate_fcs_files` and its documentation can be acessed directly from R. Note that this function assumes that the files are all identical panel-wise, i.e. they have the same parameter names and descriptions. Please use the [panel editor](#panel-editing-and-renaming) before concatenation if that is not the case.

## Bead-based normalization

The idea behind the method is described in [this](https://www.ncbi.nlm.nih.gov/pubmed/23512433) publication. This software represents an R re-implementation of the [original](https://github.com/nolanlab/bead-normalization) normalization software developed for Matlab.

Sample data for testing is available [here](https://github.com/nolanlab/bead-normalization/tree/master/sample_data) (only download the FCS in the top level directory, not the contents of the beads and normed sub-folders).

## Inputs and outputs

The normalization workflow involves the following steps:

1. Beads identification through gating
2. Data normalization
3. Beads removal (optional)

Assuming the working directory is called *working_directory* and  contains two FCS files called *A.fcs* and *B.fcs*, at the end of the workflow the following directory structure and output files will be generated

```
working_directory
|--- A.fcs
|--- B.fcs
|--- normed
     |--- A_normalized.fcs
     |--- B_normalized.fcs
     |--- beads_before_and_after.pdf
     |--- beads_gates.json
     |--- beads_vs_time
          |--- A.pdf
          |--- B.pdf
     |--- beads_removed
          |--- A_normalized_beadsremoved.fcs
          |--- B_normalized_beadsremoved.fcs
          |--- removed_events
               |--- A_normalized_removedEvents.fcs
               |--- B_normalized_removedEvents.fcs
     |--- beads
          |--- A_beads.fcs
          |--- B_beads.fcs
```

- *A_normalized.fcs*: contains the normalized data, with an added parameter called *beadDist* representing the square root of the Mahalanobis distance of each event from the centroid of the beads population
- *beads_before_and_after.pdf*: a plot of the median intensities of the beads channels before and after normalization. This plot contains a single median value per sample. Therefore it will not be informative if you are normalizing a single sample
- *beads_gates.json*: this file contains the gates that were used to identify the beads
- *A.pdf*: a plot of the intensities of the beads channels along time, before and after normalization
- *A_normalized_beadsremoved.fcs*: the normalized data with the beads events removed
- *A_normalized_removedEevents.fcs*: the events that have been removed from the normalized data based on the Mahalanobis distance cutoff 
- *A_beads.fcs*: the beads events, as identified by gating

## Starting the GUI and selecting the working directory

You can start the normalizer GUI by typing the following commands in your R session

```
library(premessa)
normalizer_GUI()
```
This will open a new web browser window, which is used for displaying the GUI. Upon starting, a file selection window will also appear from your R session. You should use this window to navigate to the directory containing the data you want to analyze, and select any file in that directory. The directory itself will then become the working directory for the software.

To stop the software simply hit the "ESC" key in your R session.

The GUI is organized in two tabs:
- *Normalize data*: used for beads gating and data normalization 
- *Remove beads*: used for beads removal

### *Normalize data* panel

This panel contains the following controls:

- *Select beads type*: select the type of normalization beads that have been used for the experiment. Most users will select the default *Fluidigm Beads (140, 151, 153, 165, 175)*. These are the beads [sold](https://www.fluidigm.com/reagents/proteomics/201078-eq-four-element-calibration-beads--100ml) by Fluidigm. The numbers indicate the beads channels used for normalization.
- *Select FCS file*: the FCS  that is currently being visualized for gating. This dropdown will contain all the FCS files located in the working directory. The gating plots will appear under the row of buttons.
- *Select baseline for normalization*: the baseline beads intensities to be used for normalization. You can either use the median beads intensities of the FCS files that you are currently using for normalization (*Current files* option), or the median intensities of an existing set of beads files (*Existing folder of beads files*). If you select the latter a file dialog window will pop-up when you select the option. Use the window to navigate to a directory containing FCS files containing beads events only (for instance the *A_beads.fcs* file in the above example) and select one of the files. The software will then load *all* the files contained in the same directory as the file you selected. The currently selected folder will be displayed in a text box on the right. 
- *Visualize beads*: clicking this button will color in red the events that are recognized as beads events in the gating plots. Note that this is for visualization only, whether you press this button or not has no effect on the normalization process
- *Apply current gates to all files*: applies the current gates to all the files.
- *Normalize*: starts the normalization routine. When the process is completed a confirmation dialog will appear

The workflow involves cycling through all the files and adjusting the beads gates in the plot, in order to identify the beads. Only events that are included in *all* the beads gates are identified as beads. As detailed in the dialog box that is above the row of buttons, only files for which the gates have been defined will be used as input for normalization.

You can cycle back and forth between different files, as the GUI will remember the gates you have selected for each file.

 
### *Remove beads* panel

This panel has the following controls

- *Select beads type*: same as for the *Normalize data* panel: select the type of normalization beads that have been used for the experiment. 
- *Select FCS file*: select the FCS file for plotting. The dropdown will contain all the FCS files located in the *normed* sub-folder of the working directory. The plots will appear below the row of buttons. See below for a description of what the plots represent
- *Cutoff for bead removal*: the Mahalanobis distance cutoff to be used for bead removal (see below).
- *Remove beads (current file)*: use the current cutoff to remove beads from the currently selected file. When the process is completed a confirmation dialog will appear
- *Remove beads (all files)*: use the current cutoff to remove beads from all the files in the folder (i.e. all the files that are listed in the *Select FCS file* dropdown). When the process is completed a confirmation dialog will appear
 
The bead removal procedure is based on the idea of looking at the distance between each event and the centroid of the beads population, and removing all the events that are closer than a given threshold to the beads population, and therefore are likely to represent beads as opposed to true cells.

To this end, during normalization, the software calculates the square root of the Mahalanobis distance of each event from the centroid of the beads population, and records this information in the *beadDist* parameter in the FCS file with the normalized data (i.e. the *_normalized.fcs* files in the *normed* sub-folder).

During the beads removal step, all the events whose *beadDist* is less or equal than the *Cutoff for bead removal* parameter are removed from the FCS. The removed events are saved in the *removed_events* sub-folder (see above).

The plots in the bottom half of the panel help you select an appropriate cutoff. They display all the pairs of beads channels. Beads should appear as a population in the upper right corner (as they will be double-positives for all the channel pairs). The color of the points represent the distance from the beads population. You should choose a cutoff so that most of the bead events are below the cutoff, and most of the non-beads events are above it. The legend for the color scale is located above the plots.

### Differences with the Matlab Normalizer

The normalization algorithm is exactly identical to the one used in the original Matlab implementation of the normalizer. The only differences relate to the way the GUI manages the workflow, and to the organization of the output directory structure.

With the Matlab implementation, when you do gating and beads removal, you process one file at the time, and there is no way to look back at previously analyzed files, for instance for adjusting the gates.

premessa separates the two steps of the workflow. First you setup the gates, moving back and forth between the input files as needed. This is useful if, for instance, you want to use the exact same gates for all the files, because you need to visualize all the data, before you can identify gates that will work across all the files. Once the gates have been setup, all the files are normalized and the results are saved. 

You then switch to the beads removal step, once again visualizing the files back and forth as needed, until you have selected an appropriate cutoff. You can then either remove beads from a single file, or from all the files simultaneously. Because the intermediate normalization results have been saved, you can repeat the beads removal step if needed.



## De-barcoding

The idea behind the method is described in [this](https://www.ncbi.nlm.nih.gov/pubmed/25612231) publication. This software represents an R re-implementation of the [original](https://github.com/nolanlab/single-cell-debarcoder) debarcoding software developed for Matlab.

Sample data for testing is available [here](https://github.com/nolanlab/single-cell-debarcoder/tree/master/sample_files) (you only need the *.csv* and *.fcs* files)

### Operation

You can start the debarcoder GUI by typing the following commands in your R session

```
library(premessa)
debarcoder_GUI()
```

Upon launching the GUI you will have access to the following controls:

- *Current barcode key*: the CSV file containing the barcode key. By default the standard Fluidigm Cell-ID 20-Plex Barcoding key is used. If you want to select a different one, press the *Select key* button. 
- *Current FCS file*: the FCS that is currently being debarcoded. Press the *Select FCS* button to select the FCS file you want to debarcode. Upon selecting both the FCS file and the key, the preliminary debarcoding process will start immediately. After a few seconds a number of diagnostic plots will appear in the right portion of the window (see [below](#plot-types))
- *Minimum separation*: the minimum seperation between the positive and negative barcode channels that an event needs to have in order to be assigned to a sample. Events where the separation is less than this threshold are left unassigned. This filtering is done after rescaling the intensity of the barcode channels, and therefore the threshold should be a number between 0 and 1
- *Maximum Mahalanobis distance*: the maximum distance between a single cell event, and the centroid of the sample the event has been assigned to. Events with distance greather than the threshold are left unassigned. The distance is capped at 30, so the default value of this option does not apply a filter based on Mahalanobis distance. Note that in some cases (for instance when there are very few events in a sample), the Mahalanobis distance calculation may fail. In this case a diagnostic message is printed in the R console, and filtering based on Mahalanobis distance is not performed for the corresponding sample
- *Plot type*: selects the type of plot to be displayed. Please see [below](#plot-types) for a description of the plots. Depending on the plot type, a few additional controls may be displayed:
  - *Select sample*: select a specific sample for plotting. Sample names are taken from the barcode key
  - *Select x axis*: select the channel to be displayed on the x axis
  - *Select y axis*: select the channel to be displayed on the y axis
- *Plot data*: hitting this button will display the currently selected *Plot type*. A spinner loading animation will appear in the button while the plot is being generated. Note that the first plot will take much longer as the data needs to be loaded the first time, but will be cached for subsequent plots
- *Save files*: hitting this button will apply the current settings, performed the debarcoding, and save the resulting output files


Assuming the FCS file *barcoded_data.fcs* is located in the directory *dir*, and the barcode key defines 3 barcoded populations (A, B, C), the following directories and output files will be created at the end of the debarcoding process:

```
dir
|--- barcoded_data.fcs
|--- debarcoded
     |--- barcoded_data_A.fcs
     |--- barcoded_data_B.fcs
     |--- barcoded_data_C.fcs
     |--- barcoded_data_Unassigned.fcs
```



### Plot types

There are four types of visualization that allow you to inspect the results of the debarcoding process, and choose the optimal seperation and Mahalanobis distance thresholds. Each plot window is subdivided into a top and bottom section, as described below:

- *Separation*
  - *Top*: A histogram of the separation between the positive and negative barcode channels for all the events
  - *Bottom*: Barcode yields as a function of the separation threshold. As the threshold increases, the number of cells assigned to each sample decreases, and more events are left unassigned. The currently selected threshold is displayed as a vertical red line.
- *Event*
  - *Top*: Bargraph or cell yields for each sample after debarcoding, given the current settings
  - *Bottom*: Scatterplot showing the barcode channel intensities for each event, ordered on the x axis. The left plot displays the original data, arcsinh transformed. The plot on the right displays the data rescaled between 0 and 1, which is actually used for debarcoding. Both plots only displays data for the selected sample (use the *Select sample* dropdown to pick the active sample)
- *Single biaxial*
  - *Top*: Bargraph of cell yields for each sample after debarcoding, given the current settings
  - *Bottom*: Scatterplot of barcode channel intensities for the selected sample. The channels displayed on the x and y axis are selected using the dropdown menus on the left (*Select x axis*, *Select y axis*). The points are colored according to the Mahalanobis distance from the centroid of the population. The color scale is displayed on top of the graph. The values plotted are arcsinh  transformed.
- *All barcode biaxials*
  - *Top*: Bargraph of cell yields for each sample after debarcoding, given the current setting
  - *Bottom*: a matrix of scatterplots of barcode channel intensities for the selected sample. All possible combinations are displayed, and the channels represented on the rows and columns are identified by the names on the axes of the bottom and left-most plots. The plots on the diagonal display the distribution of intensity values for the corresponding channel

### De-barcoding without the GUI

All the R functions necessary to perform debarcoding can be accessed directly, and documentation is available in the standard R documentation format, accessible through the R session. The main wrapper function that executes the entire procedure is called `debarcode_fcs`. Inspecting the code of this function should give you an idea of what are the steps involved, and which functions perform each one.
