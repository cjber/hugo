+++
author = "Cillian Berragan"
title = "University of Liverpool R Markdown Templates"
date = "2019-12-12"
tags = [
    "rmarkdown",
    "latex",
    "template"
]
categories = [
    "Portfolio"
]
+++

Two R Markdown templates for use in assignments and dissertations/theses at the University of Liverpool. Template written primarily in LaTeX and packaged for use in R Markdown.

<!--more-->

---
<p align="center">
<font size="5">
<a href="https://github.com/cjber/uolrmarkdown"> Repository Link</a>
</font>
</p>

---

# Installation

Install with:

```r
devtools::install_github("cjber/uolrmarkdown")
```

If using RStudio:

`New File -> R Markdown -> From Template -> UoL R Markdown`

Otherwise for the article:

```r
rmarkdown::draft(
  'index.Rmd', template = 'article',
  package = 'uolrmarkdown', create_dir = TRUE
)
```

and for the report:

```r
rmarkdown::draft(
  'index.Rmd', template = 'report',
  package = 'uolrmarkdown', create_dir = TRUE
)
```

# Example Images

| Chapter Report | Two Column Article |
| :------------: | :----------------: |
![](https://raw.githubusercontent.com/cjber/uolrmarkdown/master/chapter.png) | ![](https://raw.githubusercontent.com/cjber/uolrmarkdown/master/twocol.png)

# Customisation

Customisation of the knitr chunks can be accessed through `template -> defaults.r`. Here additional citations may be manually added if required e.g. QGIS. Additionally the default ggplot2 theme is set here.

`scripts -> functions.r` provides an example r script for loading in required packages and defining custom functions. I generally use `source("./functions.r")` with my scripts to prevent loading the required libs and functions each time.

The main bib file is given in `bib/kbib.bib` I use Zotero to generate bibtex.

For the article template for the written work only the main `.Rmd` needs editing, but in the chapter report each individual chapter should be edited separately.

For additional help with R Markdown see [yihui](https://bookdown.org/yihui/rmarkdown/).
