
<!-- README.md is generated from README.Rmd. Please edit that file -->

# rtauargus <a href='https://inseefrlab.github.io/rtauargus/'><img src='man/figures/rtauargus_logo_small.png' align="right" width="120" /></a>

The package documentation is available [here](https://inseefrlab.github.io/rtauargus/).

<!-- badges: start -->
<!-- [![pipeline status](https://gitlab.insee.fr/outilsconfidentialite/rtauargus/badges/master/pipeline.svg)](https://gitlab.insee.fr/outilsconfidentialite/rtauargus/-/pipelines) -->
<!-- badges: end -->
<!--![](vignettes/R_logo_small.png) ![](vignettes/TauBall2_small.png)-->

## Run τ-Argus from R

The *rtauargus* package provides an **R** interface for **τ-Argus**.

It allows to:

- create inputs (rda, arb, hst and tab files) from data in R format ;
- generate the sequence of instructions to be executed in batch mode
  (arb file);
- launch a τ-Argus batch in command line;
- retrieve the results in R.

These different operations can be executed in one go, but also in a
modular way. They allow to integrate the tasks performed by τ-Argus in a
processing chain written in R.

The package presents other **additional functionalities**, such as:

- managing the protection of several tables at once;
- creating a hierarchical variable from correspondence table.

It’s possible to choose a tabular or microdata approach, but the tabular
one is, from now on, encouraged.

## Installation

- **most recent stable version** (recommended)

  - For Insee agents:

    ``` r
    install.packages(
      "rtauargus",
      repos = "https://nexus.insee.fr/repository/r-public",
      type = "source"
    )
    ```

  - Elsewhere:

    ``` r
    install.packages("remotes")
    remotes::install_github(
      "InseeFrLab/rtauargus",
      build_vignettes = FALSE,
      upgrade = "never"
    )
    ```

- **version in development**

To install a specific version, add to the directory a reference
([commit](https://github.com/inseefrlab/rtauargus/commits/master) or
[tag](https://github.com/inseefrlab/rtauargus/tags)), for example
`"inseefrlab/rtauargus@v-0.4.1"`.

## Simple example

When loading the package, the console displays some information:

``` r
library(rtauargus)
```

In particular, a plausible location for the τ-Argus software is
predefined. This can be changed for the duration of the R session, as
follows:

``` r
loc_tauargus <- "Y:/Logiciels/TauArgus/TauArgus4.2.2b1/TauArgus.exe"
options(rtauargus.tauargus_exe = loc_tauargus)
```

With this small adjustment done, the package is ready to be used.

For the following demonstration, a fictitious table will be used:

``` r
act_size <-
  data.frame(
    ACTIVITY = c("01","01","01","02","02","02","06","06","06","Total","Total","Total"),
    SIZE = c("tr1","tr2","Total","tr1","tr2","Total","tr1","tr2","Total","tr1","tr2","Total"),
    VAL = c(100,50,150,30,20,50,60,40,100,190,110,300),
    N_OBS = c(10,5,15,2,5,7,8,6,14,20,16,36),
    MAX = c(20,15,20,20,10,20,16,38,38,20,38,38)
  )
```

As primary rules, we use the two following ones:

- The n-k dominance rule with n=1 and k = 85
- The minimum frequency rule with n = 3 and a safety range of 10.

To get the results for the dominance rule, we need to specify the
largest contributor to each cell, corresponding to the `MAX` variable in
the tabular data.

``` r
ex1 <- tab_rtauargus(
  act_size,
  dir_name = "tauargus_files",
  files_name = "ex1",
  explanatory_vars = c("ACTIVITY","SIZE"),
  safety_rules = "FREQ(3,10)|NK(1,85)",
  value = "VAL",
  freq = "N_OBS",
  maxscore = "MAX",
  totcode = c(ACTIVITY="Total",SIZE="Total")
)
#> Start of batch procedure; file: Z:\SDC\OutilsConfidentialite\rtauargus\tauargus_files\ex1.arb
#> <OPENTABLEDATA> "Z:\SDC\OutilsConfidentialite\rtauargus\tauargus_files\ex1.tab"
#> <OPENMETADATA> "Z:\SDC\OutilsConfidentialite\rtauargus\tauargus_files\ex1.rda"
#> <SPECIFYTABLE> "ACTIVITY""SIZE"|"VAL"||
#> <SAFETYRULE> FREQ(3,10)|NK(1,85)
#> <READTABLE> 1
#> Tables have been read
#> <SUPPRESS> MOD(1,5,1,0,0)
#> Start of the modular protection for table ACTIVITY x SIZE | VAL
#> End of modular protection. Time used 0 seconds
#>                    Number of suppressions: 2
#> <WRITETABLE> (1,4,,"Z:\SDC\OutilsConfidentialite\rtauargus\tauargus_files\ex1.csv")
#> Table: ACTIVITY x SIZE | VAL has been written
#>                    Output file name: Z:\SDC\OutilsConfidentialite\rtauargus\tauargus_files\ex1.csv
#> End of TauArgus run
```

By default, the function displays in the console the logbook content in
which user can read all steps run by τ-Argus. This can be retrieved in
the logbook.txt file. With `verbose = FALSE`, the function can be
silenced.

By default, the function returns the original dataset with one variable
more, called `Status`, directly resulting from τ-Argus and describing
the status of each cell as follows:

\-`A`: primary secret cell because of frequency rule;  
-`B`: primary secret cell because of dominance rule (1st contributor);  
-`C`: primary secret cell because of frequency rule (more contributors
in case when n\>1);  
-`D`: secondary secret cell;  
-`V`: valid cells - no need to mask.

``` r
ex1
#>    ACTIVITY  SIZE VAL N_OBS MAX Status
#> 1        01 Total 150    15  20      V
#> 2        01   tr1 100    10  20      V
#> 3        01   tr2  50     5  15      V
#> 4        02 Total  50     7  20      V
#> 5        02   tr1  30     2  20      A
#> 6        02   tr2  20     5  10      D
#> 7        06 Total 100    14  38      V
#> 8        06   tr1  60     8  16      D
#> 9        06   tr2  40     6  38      B
#> 10    Total Total 300    36  38      V
#> 11    Total   tr1 190    20  20      V
#> 12    Total   tr2 110    16  38      V
```

All the files generated by the function are written in the specified
directory (`dir_name` argument). The default format for the protected
table is csv but it can be changed. All the τ-Argus files (.tab, .rda,
.arb and .txt) are written in the same directory, too. To go further,
you can consult the latest version of the τ-Argus manual is downloadable
here: <https://research.cbs.nl/casc/Software/TauManualV4.1.pdf>.

**A detailed overview is available via `vignette("rtauargus")`.**

## Important notes

The functions of *rtauargus* calling τ-Argus require that this software
be accessible from the workstation. The download of τ-Argus is done on
the [dedicated page](https://github.com/sdcTools/tauargus/releases) of
the *sdcTools* git repository.

\_The package was developed on the basis of open source versions of
τ-Argus (versions 4.2 and above), in particular the version used for
this version is τ-Argus 4.2.3. It is not compatible with version
3.5.\*\*\_
