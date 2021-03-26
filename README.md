# GeneScopeFK: FastK compatible version of GenomeScope 2.0
 
<font size ="4">**_Authors:  Gene Myers**<br>
**_First:   Mar 18, 2021_**<br>
**_Current: Feb 18, 2021_**</font>


This is a stripped down version of [GenomeScope2.0](https://github.com/tbenavi1/genomescope2.0) that operates on customized
histograms that can be produced by [FastK](https://github.com/thegenemyers/FASTK).  The package contains just the components needed
to build the command line version.

The GenomeScope2.0 model can be inferred very accurately with a histogram of just the
first several hundred k-mer frequencies.  Unfortunately, to accurately estimate the genome size from this model, as currently configured,
GenomeScope2.0 requires a complete, unbounded histogram, i.e. every frequency that has a non-zero
count.  The reason for this
dependency is that GenomeScope2.0 computes the total # of base pairs of data as the sum of the
product of each frequency with its count.  This requirment unduly taxes k-mer
counters, forcing them to keep track of counts into the 100s of thousands, just
in order to know how many k-mers there were in the original data set.
Moreover, FastK only counts up to 32,767 (a 2-byte count) but does keep track of the total
counts of all k-mers including those with counts greater than this limit.

So our slight revision of GenomeScope2.0, that we call **GeneScopeFK**, takes a histogram where the last entry of the
histogram is such that the product of its frequency, say F, with its count is approximately equal to the what would be the sum of the products of all the frequencies greater
than or equal to F in a complete histogram.  The estimate is off by at most F-1
counts.  GeneScopeFK uses all but the last hisotgram entry for the model fitting
and then the entire histogram with its faux last entry to estimate genome size.

Given a FastK table for a dataset, Histex with the -G option produces such a
histogram.  The command sequence below produces GenomeScope output in subdirectory
`Output`.

```
  FastK -v -t1 -k<K> <dataset> -NTable
  Histex -G Table | GeneScopeFK -o Output -k <K>
```
  
### Installing GeneScopeFK for the Command Line

Get the package by either downloading it or cloning this repository.

It is assumed you have installed and up to date version of R and also have
installed the packages "minpack.lm" and "argparse".

If you already have write access to a specific path for R libraries that your R installation is set up to use, please set the local_lib_path
variable in the install.R file equal to this path. Then we can run:

    $ Rscript install.R
    
For most users, it is instead easiest to install GeneScopeFK in a local directory. In this example the user creates an "R_libs" folder located in the home directory to use for local R libraries:

    $ mkdir ~/R_libs
    
In order for R to load libraries from this location, the user then has to create and/or edit the .Renviron file in the home directory:

    $ echo "R_LIBS=~/R_libs/" >> ~/.Renviron

Now one can install GenomeScope 2.0:

    $ Rscript install.R

After installation, command line users can run the modeling with the R script GeneScopeFK.R, making sure that GeneScope.R is on your PATH.

    $ GeneScopeFK.R -i <histogram_file> -o <output_dir> -k <K>
