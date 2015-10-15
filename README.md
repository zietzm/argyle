# <img src="figs/argyle.png" alt="argyle logo" height=50 /> argyle
**An R package for GenotYpes from ILlumina Et al.**
Utilities for import, QC and (some) analysis of genotyping and hybridization-intensity data from Illumina Infinium arrays using `R`.

## Dependencies
Effort has been made to keep to a minimum the number of package dependencies, subject to the constraint that I don't want to re-implement from scratch what others have done better.

* `data.table`: really fast and efficient handling of big (multi-GB scale) table-style data with low overhead
* `preprocessCore` (Biodoncuctor): robust quantile normalization routine written in `C`
* `plyr`: generalizations of base-`R`'s `apply()` family
* `reshape2`: easy "flattening" of matrices to dataframes
* `digest`: for computing MD5 checksums to check data integrity

The following are required for some functions but one could get by without them:

* `ggplot2` (and friends): required for the plotting functions
* `corpcor`: required for "fast"-mode PCA
* `Matrix`: sparse matrices for LD calculations

## Installation
Installation of the package directly from Github requires `devtools`.
```
library(devtools)

## allow R to look for pacakges in both CRAN and Bioconductor
setRepositories(ind = 1:2)

## install from Github source
install_github("andrewparkermorgan/argyle")
```


## Usage
```
## load example dataset
data(ex)

## high-level summary
summary(ex)

## NB: print(.) same as summary, won't flood the terminal
print(ex)

## peek at genotype matrix
head(ex)

## see marker map and sample metadata
markers(ex)
samples(ex)

## subset operations: hard brackets or subset()
ex[ 1:10,1:2 ]

## or stuff like
x <- subset(ex, chr == "chrM")
x <- subset(ex, sex == 2, by = "samples")

## run QC checks and flag samples above thresholds
ex <- run.sample.qc(ex, max.H = 5e3, max.N = 500)
# how many samples fail QC?
summarize.filters(ex)
```

## Interface to [`R/DOQTL`](http://cgd.jax.org/apps/doqtl/DOQTL.shtml)
Genotypes processed with `argyle` can be packaged into a set of `R` objects (bundled in an `*.Rdata` file) suitable for use as input to Dan Gatti's `DOQTL` software.  `DOQTL` performs haplotype reconstruction and genetic mapping (under both linkage/composite-interval and single-marker association models) in multifounder advanced intercross populations.  Its namesake is the Diversity Outbred (DO) mouse population (see [do.jax.org](http://do.jax.org/)).
```
## export for DOQTL
export.doqtl(ex, "./doqtl.objects.Rdata")

## convert to R/qtl
cross <- as.rqtl(ex, type = "f2")
```

## Interface to [`PLINK`](https://www.cog-genomics.org/plink2/)
Computation on large SNP array genotyping datasets is not a new problem.  Many common operations -- frequency statistics (sample-wise and marker-wise), differentiation statistics ($F_{st}$ et al), homozygosity checks, association testing, multivariate clustering by PCA and MDS -- are implemented efficiently in the `PLINK` package.  The input formats popularized by `PLINK` are now used by other software in population genetics.

This package provides functions to read and write **binary** `PLINK` filesets.  The binary fileset consists of three files:

* `*.fam`: the ``family file'' describing samples (6 columns): family ID, sample ID, mom ID, dad ID, sex (0=unknown, 1=male, 2=female), phenotype (-9=missing)
* `*.bim`: a file describing the marker map (6 columns, at least): chromosome, marker ID, genetic position (cM), physical position (bp), allele 1, allele 2
* `*.bed`: compact binary representation of genotypes using 2 bits per genotype

Note that order matters: genotypes from the `*.bed` file are mapped to samples and markers using order of appearance in the `*.fam` and `*.bim` files.
```
## this command produces files 'sample.bed', 'sample.bim' and 'sample.fam' in the R sessions temporary directory
ff <- file.path(tempdir(), "sample")
ex <- recode(ex, "native")
write.plink(ex, ff)

## ... and this one reads it back in
summary( read.plink(ff) )
```

Also included are thin wrappers around many `PLINK` utilities.  These of course require a working executable named `plink` in the user's path. See the user manual for details.