Data Load
================
Abi Finn
2024-07-05

# Data Loading and Merging Steps

# Dataset loading

This demonstrates how to quickly load the Synthetic Population,
Understanding Society (US) survey, and geo lookup.

- This section also demonstrates how data can be reduced quickly and
  merged to fully set up the Synthetic Population.

## Load the Synthetic Population

``` r
# load sp # 
sp_8cons <- data.table::fread("RData/sp_ind_wavek_census2011_est2020_8cons.csv")

# explore 
str(sp_8cons)      # general structure 
```

    ## Classes 'data.table' and 'data.frame':   52853971 obs. of  2 variables:
    ##  $ ZoneID: chr  "E01000001" "E01000001" "E01000001" "E01000001" ...
    ##  $ pidp  : num  5.44e+08 9.57e+08 1.57e+09 4.77e+08 5.45e+08 ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

``` r
sp_8cons[1:5,]     # first 5 rows of all columns 
```

    ##       ZoneID       pidp
    ##       <char>      <num>
    ## 1: E01000001  544320967
    ## 2: E01000001  957059290
    ## 3: E01000001 1565220611
    ## 4: E01000001  476964927
    ## 5: E01000001  544752771

## Subset for Geography of Interest

This bit of code uses the geography of interest defined previously.

- In the definitions section of *SynthPop_dataPrep.md* file.

- *definitions\$geo_subset*

``` r
load("variables.RData")


sp_8cons[, ctr_code := substr(ZoneID, 1, 1)] # first letter of Zone ID
sp_8cons <- sp_8cons[ctr_code %in% definitions$geo_subset, ]
```

## Load Understanding Society (US) data

``` r
us_ind <- data.table::fread("RData/k_indresp.tab")
dim(us_ind)      # general structure
```

    ## [1] 32008  3258

``` r
us_ind[1:5,1:5]  # first 5 rows of the first 5 variables
```

    ##        pidp   pid   k_hidp k_pno k_hhorig
    ##       <int> <int>    <int> <int>    <int>
    ## 1: 68006127    -8 68013620     1        1
    ## 2: 68020564    -8 68013620     2        1
    ## 3: 68008847    -8 68027220     1        1
    ## 4: 68009527    -8 68034020     1        1
    ## 5: 68061288    -8 68034020     2        1

## Rename columns in US dataset

Get rid of the variables that start with ‘k’

- Columns in this dataset have this prefix to denote the ‘wave’ of the
  US survey data that they belong to.

``` r
data.table::setnames(us_ind, old = names(us_ind),
              new = gsub(paste0("^",definitions$us_wave,"_"),"",names(us_ind)))
us_ind[1:5,1:5]  # first 5 rows of the first 5 variables
```

    ##        pidp   pid     hidp   pno hhorig
    ##       <int> <int>    <int> <int>  <int>
    ## 1: 68006127    -8 68013620     1      1
    ## 2: 68020564    -8 68013620     2      1
    ## 3: 68008847    -8 68027220     1      1
    ## 4: 68009527    -8 68034020     1      1
    ## 5: 68061288    -8 68034020     2      1

## Select variables to keep from the US dataset

This is where you can specify which variables you want to keep.

If you aren’t familiar with the variables contained in the US dataset,
please refer to this
[link](https://www.understandingsociety.ac.uk/documentation/mainstage/user-guides/main-survey-user-guide/understanding-society-key-variables/).

``` r
us_ind_keep_var <- c("pidp", "hidp",  # id for individuals
                     "sex", "age_dv", # age and sex
                     "sf12mcs_dv",    # mental health indicator SF-12
                     "employ")        # employment
# apply selection 
us_ind <- us_ind[, ..us_ind_keep_var]
```

## Load US raw data for households

``` r
us_hh <- data.table::fread("RData/k_hhresp.tab")
dim(us_hh)       # general structure
```

    ## [1] 18139   319

``` r
us_hh[1:5,1:5]   # first 5 rows of the first 5 variables
```

    ##      k_hidp k_intnum k_hhorig k_psu k_strata
    ##       <int>    <int>    <int> <int>    <int>
    ## 1: 68013620       -8        1  2012     2006
    ## 2: 68027220       -8        1  2012     2006
    ## 3: 68034020       -8        1  2012     2006
    ## 4: 68040820       -8        1  2012     2006
    ## 5: 68054420 21008765        1  2036     2018

## Rename (remove k prefix)

``` r
data.table::setnames(us_hh, old = names(us_hh),
                     new = gsub(paste0("^",definitions$us_wave,"_"),"",names(us_hh)))
us_hh[1:5,1:5]  # first 5 rows of the first 5 variables
```

    ##        hidp   intnum hhorig   psu strata
    ##       <int>    <int>  <int> <int>  <int>
    ## 1: 68013620       -8      1  2012   2006
    ## 2: 68027220       -8      1  2012   2006
    ## 3: 68034020       -8      1  2012   2006
    ## 4: 68040820       -8      1  2012   2006
    ## 5: 68054420 21008765      1  2036   2018

## Select variables to keep from this dataset

``` r
# us_hh_keep_var <- c("hidp",    # id for hh to link with individuals 
#                     "xphsdct") # financial hardship: problems paying council tax 

us_hh_keep_var <- c("hidp",    # id for hh to link with individuals 
                    "fihhmngrs_dv") # gross household income: month before interview

# apply selection
us_hh <- us_hh[, ..us_hh_keep_var]
```

# Merging of datasets

## Merge the Synthetic Pop with the US individual data

``` r
sp_8cons <- merge(sp_8cons, us_ind, by = "pidp", all.x = TRUE) 
data.table::setkey(sp_8cons, ZoneID)
sp_8cons[1:5,]     # check result first 10 rows of all columns 
```

    ## Key: <ZoneID>
    ##        pidp    ZoneID ctr_code      hidp   sex age_dv sf12mcs_dv employ
    ##       <int>    <char>   <char>     <int> <int>  <int>      <num>  <int>
    ## 1:   599765 E01000001        E 210528700     2     32      57.16      1
    ## 2:  1587125 E01000001        E 617671220     2     53      31.92      1
    ## 3: 68029931 E01000001        E  68102020     1     50      54.20      1
    ## 4: 68064605 E01000001        E  72454020     1     70      58.39      2
    ## 5: 68133969 E01000001        E  72488020     1     85      55.18      2

## Merge the Synthetic Pop with the US household data

``` r
# merge synthetic population with US data for households  
sp_8cons <- merge(sp_8cons, us_hh, by = "hidp", all.x = TRUE)  

# almost there! 
sp_8cons
```

    ## Key: <hidp>
    ##                 hidp       pidp    ZoneID ctr_code   sex age_dv sf12mcs_dv
    ##                <int>      <int>    <char>   <char> <int>  <int>      <num>
    ##        1:   68013620   68020564 E01000010        E     1     48      33.70
    ##        2:   68013620   68020564 E01000012        E     1     48      33.70
    ##        3:   68013620   68020564 E01000013        E     1     48      33.70
    ##        4:   68013620   68006127 E01000034        E     2     49      54.78
    ##        5:   68013620   68020564 E01000040        E     1     48      33.70
    ##       ---                                                                 
    ## 45697894: 1637766420 1632682051 E01033428        E     2     58      44.44
    ## 45697895: 1637766420 1632682051 E01033432        E     2     58      44.44
    ## 45697896: 1637766420 1632682051 E01033448        E     2     58      44.44
    ## 45697897: 1637766420 1632682051 E01033723        E     2     58      44.44
    ## 45697898: 1637766420 1632682051 E01033739        E     2     58      44.44
    ##           employ fihhmngrs_dv
    ##            <int>        <num>
    ##        1:      2      1565.67
    ##        2:      2      1565.67
    ##        3:      2      1565.67
    ##        4:      2      1565.67
    ##        5:      2      1565.67
    ##       ---                    
    ## 45697894:      1      8078.94
    ## 45697895:      1      8078.94
    ## 45697896:      1      8078.94
    ## 45697897:      1      8078.94
    ## 45697898:      1      8078.94

# Load geo lookup and merge with SP

<p>

A geo-lookup is a file which describes the hierarchy of geographies.

- e.g., LSOAs –\> MSOA –\> GOR –\> Countries –\> UK

- [An example of where you can get
  these.](https://geoportal.statistics.gov.uk/)

  </p>

## Load geo lookup

``` r
lookup_oa <- data.table::fread("RData/OA_to_Local_Authority_District_May_2021.csv")
str(lookup_oa)
```

    ## Classes 'data.table' and 'data.frame':   2661131 obs. of  14 variables:
    ##  $ pcd7    : chr  "AB1 0AA" "AB1 0AB" "AB1 0AD" "AB1 0AE" ...
    ##  $ pcd8    : chr  "AB1  0AA" "AB1  0AB" "AB1  0AD" "AB1  0AE" ...
    ##  $ pcds    : chr  "AB1 0AA" "AB1 0AB" "AB1 0AD" "AB1 0AE" ...
    ##  $ dointr  : int  198001 198001 198001 199402 199012 199012 198001 198001 198001 198001 ...
    ##  $ doterm  : int  199606 199606 199606 199606 199207 199207 199606 199606 199606 199606 ...
    ##  $ usertype: int  0 0 0 0 1 1 0 0 1 0 ...
    ##  $ oa11cd  : chr  "S00090303" "S00090303" "S00090399" "S00091322" ...
    ##  $ lsoa11cd: chr  "S01006514" "S01006514" "S01006514" "S01006853" ...
    ##  $ msoa11cd: chr  "S02001237" "S02001237" "S02001237" "S02001296" ...
    ##  $ ladcd   : chr  "S12000033" "S12000033" "S12000033" "S12000034" ...
    ##  $ lsoa11nm: chr  "Cults, Bieldside and Milltimber West - 02" "Cults, Bieldside and Milltimber West - 02" "Cults, Bieldside and Milltimber West - 02" "Dunecht, Durris and Drumoak - 01" ...
    ##  $ msoa11nm: chr  "Cults, Bieldside and Milltimber Wes" "Cults, Bieldside and Milltimber Wes" "Cults, Bieldside and Milltimber Wes" "Dunecht, Durris and Drumoak" ...
    ##  $ ladnm   : chr  "Aberdeen City" "Aberdeen City" "Aberdeen City" "Aberdeenshire" ...
    ##  $ ladnmw  : chr  "" "" "" "" ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

## Reduce geo lookup

``` r
lookup_oa <- unique(lookup_oa[, .(lsoa11cd, msoa11cd, ladcd, ladnm)])
str(lookup_oa)
```

    ## Classes 'data.table' and 'data.frame':   42632 obs. of  4 variables:
    ##  $ lsoa11cd: chr  "S01006514" "S01006853" "S01006511" "S01006506" ...
    ##  $ msoa11cd: chr  "S02001237" "S02001296" "S02001236" "S02001236" ...
    ##  $ ladcd   : chr  "S12000033" "S12000034" "S12000033" "S12000033" ...
    ##  $ ladnm   : chr  "Aberdeen City" "Aberdeenshire" "Aberdeen City" "Aberdeen City" ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

## Merge geo lookup file with Synth Pop

``` r
sp_8cons <- merge(sp_8cons, lookup_oa, by.x = "ZoneID", by.y = "lsoa11cd",
                  all.x = TRUE)  
```

# Sort Columns and Check Final Dataset

## Check and sort data

``` r
# check data set before reordering
sp_8cons[1:5,]  
```

    ## Key: <ZoneID>
    ##       ZoneID     hidp     pidp ctr_code   sex age_dv sf12mcs_dv employ
    ##       <char>    <int>    <int>   <char> <int>  <int>      <num>  <int>
    ## 1: E01000001 68102020 68029931        E     1     50      54.20      1
    ## 2: E01000001 68455620 68538656        E     1     32      52.08      1
    ## 3: E01000001 68537220 68155063        E     1     26      54.10      1
    ## 4: E01000001 68741220 68197887        E     2     58      57.33      1
    ## 5: E01000001 68768420 68202647        E     2     44      40.77      1
    ##    fihhmngrs_dv  msoa11cd     ladcd          ladnm
    ##           <num>    <char>    <char>         <char>
    ## 1:      4352.19 E02000001 E09000001 City of London
    ## 2:      4083.93 E02000001 E09000001 City of London
    ## 3:      6895.80 E02000001 E09000001 City of London
    ## 4:      1449.83 E02000001 E09000001 City of London
    ## 5:      9203.70 E02000001 E09000001 City of London

``` r
# sort columns: order geography levels and IDs 
data.table::setcolorder(sp_8cons, neworder = c("ZoneID", "msoa11cd", "ladcd",
                                        "ladnm", "ctr_code",  "pidp", "hidp"))
```

## Final data check

``` r
sp_8cons[1:5,]  
```

    ## Key: <ZoneID>
    ##       ZoneID  msoa11cd     ladcd          ladnm ctr_code     pidp     hidp
    ##       <char>    <char>    <char>         <char>   <char>    <int>    <int>
    ## 1: E01000001 E02000001 E09000001 City of London        E 68029931 68102020
    ## 2: E01000001 E02000001 E09000001 City of London        E 68538656 68455620
    ## 3: E01000001 E02000001 E09000001 City of London        E 68155063 68537220
    ## 4: E01000001 E02000001 E09000001 City of London        E 68197887 68741220
    ## 5: E01000001 E02000001 E09000001 City of London        E 68202647 68768420
    ##      sex age_dv sf12mcs_dv employ fihhmngrs_dv
    ##    <int>  <int>      <num>  <int>        <num>
    ## 1:     1     50      54.20      1      4352.19
    ## 2:     1     32      52.08      1      4083.93
    ## 3:     1     26      54.10      1      6895.80
    ## 4:     2     58      57.33      1      1449.83
    ## 5:     2     44      40.77      1      9203.70

``` r
str(sp_8cons) 
```

    ## Classes 'data.table' and 'data.frame':   45697898 obs. of  12 variables:
    ##  $ ZoneID      : chr  "E01000001" "E01000001" "E01000001" "E01000001" ...
    ##  $ msoa11cd    : chr  "E02000001" "E02000001" "E02000001" "E02000001" ...
    ##  $ ladcd       : chr  "E09000001" "E09000001" "E09000001" "E09000001" ...
    ##  $ ladnm       : chr  "City of London" "City of London" "City of London" "City of London" ...
    ##  $ ctr_code    : chr  "E" "E" "E" "E" ...
    ##  $ pidp        : int  68029931 68538656 68155063 68197887 68202647 68231211 69013378 68278127 68336615 68361771 ...
    ##  $ hidp        : int  68102020 68455620 68537220 68741220 68768420 68904420 68938420 69020020 69217220 69353220 ...
    ##  $ sex         : int  1 1 1 2 2 1 1 2 1 2 ...
    ##  $ age_dv      : int  50 32 26 58 44 52 70 72 31 43 ...
    ##  $ sf12mcs_dv  : num  54.2 52.1 54.1 57.3 40.8 ...
    ##  $ employ      : int  1 1 1 1 1 1 2 2 1 1 ...
    ##  $ fihhmngrs_dv: num  4352 4084 6896 1450 9204 ...
    ##  - attr(*, ".internal.selfref")=<externalptr> 
    ##  - attr(*, "sorted")= chr "ZoneID"
