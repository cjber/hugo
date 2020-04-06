+++
author = "Cillian Berragan"
title = "Managing References in R Markdown and Rnoweb"
date = "2019-11-15"
tags = [
    "rmarkdown",
    "rnoweb",
    "latex",
    "bibtex",
    "R",
]

categories = [
    "Blog"
]
+++

When writing my dissertation I found it difficult to dynamically manage references to R packages that I had used along the way. For the main document refernces I use Zotero which makes managing these trivial, but for packages, it wasn't quite a simple.

<!--more-->

## Solution

To prevent loading in all R packages required for each script I created for my dissertation, I decided to create a central script containing them all, along with functions I had created. Within this central script, using the R package `pacman` I was able to install and load packages as required with one command;

```r
library(pacman)
pkgs <- c(
  "sf",
  "tidyverse",
  "lidR",
  "kableExtra",
  "bibtex"
)
pacman::p_load(pkgs, character.only = T)
```

Given the object `pkgs` now provides a list of all packages used in the analysis, this can be used to extract all the required references. In the main document:

```r
source("./scripts/central_script.r")
write.bib(pkgs, "rbib.bib")
cat(paste0(
  "\\nocite{",
  paste0(pkgs, collapse = ",", sep = ""), "}"
))
```

The output of this code is a `.bib` file containing all references to packages loaded, and in order to add each package to the bibliography without directing citing, the chunk adds the LaTex command `\nocite{}` containing all packges.
