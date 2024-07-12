Data Preparation
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
setwd("path/to/files")
```

# Clean Workspace

``` r
rm(list = ls())   # remove all objects from work space 
gc(full = TRUE)   # deep clean garbage
dir()             # check R has picked up relative file paths 
```

# Load Relevant Libraries

``` r
# List of required packages 
RequiredPackages <- c("data.table",  # for dialect 
                      "ggplot2",     # for plotting
                      "sf", "scales") # for maps

# Ensure all packages are installed and loaded 
.EnsurePackages <- function(packages_vector) {
  new_package <- packages_vector[!(packages_vector %in% 
                                     installed.packages()[, "Package"])]
  if (length(new_package) != 0) {
    install.packages(new_package) }
  sapply(packages_vector, suppressPackageStartupMessages(require),
         character.only = TRUE)
}
.EnsurePackages(RequiredPackages)
```

    ## Loading required package: data.table

    ## Loading required package: ggplot2

    ## Loading required package: sf

    ## Linking to GEOS 3.12.1, GDAL 3.8.4, PROJ 9.3.1; sf_use_s2() is TRUE

    ## Loading required package: scales

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
