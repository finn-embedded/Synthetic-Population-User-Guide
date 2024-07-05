SynthPop_dataPrep
================
Abi Finn
2024-07-05

# About

# Set Working Directory

- Setting the working directory ensures the script will read files from
  a specific location or write files to a specific location, without you
  specifying the fullpath.

- It should contain sub-folders for data, scripts, and output.

``` r
setwd("C:/Users/elizabeth.finn/OneDrive - Greater Manchester Combined Authority/SIPHER/syntheticPopulation/scripts/workshopVersions")
```

# Clean Workspace

# Load Relevant Libraries

# Definitions

- The following code will set out key definitions of variables that will
  be used in later scripts.

- **Important to note: This is where you can change the area you are
  interested in within the Synthetic Population.**

  - For example, you can change it from England (‘E’) to Wales (‘W’).

``` r
# Initialize definitions ...
definitions <- list()

# Start benchmarking
definitions$run_start <- Sys.time()

# Specify survey wave and country subset 
definitions$us_wave          <- c("k")   # "k" - US wave, do not change for now
definitions$geo_subset       <- c("E")   # "E" - for England (or "W" for England)
# might take a bit longer for England or England and Wales combined 

# Configure visuals  
definitions$fig_width  <- 25   # = width of output figures in cm
definitions$fig_height <- 15   # = height of output figures in cm
definitions$fig_unit   <- "cm" # = height of output figures in cm
definitions$map_cols   <- c("seagreen","lightgrey","salmon")
```
