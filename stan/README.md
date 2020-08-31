
# `drake` R package Stan model example

The goal of this workflow is to validate a small Bayesian hierarchical
model.

``` r
y_i ~ iid Normal(alpha + x_i * beta, sigma^2)
alpha ~ Normal(0, 1)
beta ~ Normal(0, 1)
sigma ~ Uniform(0, 1)
```

We simulate multiple datasets from the model and fit the model on each
dataset. For each model fit, we determine if the 50% credible interval
of the regression coefficient `beta` contains the true value of `beta`
used to generate the data. If we implemented the model correctly,
roughly 50% of the models should recapture the true `beta` in 50%
credible intervals.

## The `drake` pipeline

The [`drake`](https://github.com/ropensci/drake) R package manages the
workflow. It automatically skips steps of the pipeline when the results
are already up to date, which is critical for Bayesian data analysis
because it usually takes a long time to run Markov chain Monte Carlo. It
also helps users understand and communicate this work dependency graphs
(see `r_vis_drake_graph()`).

## File structure

The files in this example are organized as follows.

``` r
├── run.sh
├── run.R
├── _drake.R
├── sge.tmpl
├── R/
├──── packages.R
├──── functions.R
├──── plan.R
├── stan/
├──── model.stan
└── report.Rmd
```

| File              | Purpose                                                                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `run.sh`          | Shell script to run `run.R` in a persistent background process. Works on Unix-like systems. Helpful for long computations on servers. |
| `run.R`           | R script to run `r_make()`.                                                                                                           |
| `_drake.R`        | The special R script that powers functions `r_make()` and friends ([details here]()).                                                 |
| `sge.tmpl`        | A [`clustermq`](https://github.com/mschubert/clustermq) template file to deploy targets in parallel to a Sun Grid Engine cluster.     |
| `R/packages.R`    | A custom R script loading the packages we need.                                                                                       |
| `R/functions.R`   | A custom R script with user-defined functions.                                                                                        |
| `R/plan.R`        | A custom R script that defines the `drake` plan.                                                                                      |
| `stan/model.stan` | The specification of our Stan model.                                                                                                  |
| `report.Rmd`      | An R Markdown report summarizing the results of the analysis.                                                                         |

## How to run

1.  Install the packages mentioned in `R/packages.R`.
2.  Run the `drake` pipeline by either running `run.R` or `run.sh`. (The
    latter is for Unix-like systems only). This computation could take a
    while.
3.  View the validation results in the output `report.html` file.
4.  Make changes to the R code or Stan model, rerun the pipeline, and
    watch `drake` skip steps that are already up to date.

## Scale out

This computation is currently downsized for pedagogical purposes. To
scale it up, open the `R/plan.R` script and increase the number of
simulations (the number inside `seq_len()` in the `index` target).

## High-performance computing

You can run this project locally on your laptop or remotely on a
cluster. The comments in the `_drake.R` file have specific directions.
Details on high-performance computing are available in [this chapter of
the manual](https://books.ropensci.org/drake/hpc.html).
