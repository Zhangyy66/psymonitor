
<!-- README.md is generated from README.Rmd. Please edit that file -->

# psymonitor <img src="man/figures/logo.png" align="right" height=139/>

[![lifecycle](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![Travis build
status](https://travis-ci.org/itamarcaspi/psymonitor.svg?branch=master)](https://travis-ci.org/itamarcaspi/psymonitor)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/itamarcaspi/psymonitor?branch=master&svg=true)](https://ci.appveyor.com/project/itamarcaspi/psymonitor)
[![CRAN
status](https://www.r-pkg.org/badges/version/psymonitor)](https://cran.r-project.org/package=psymonitor)
[![metacran
downloads](https://cranlogs.r-pkg.org/badges/psymonitor)](https://cran.r-project.org/package=psymonitor)

`psymonitor` provides an accessible implementation of the popular
real-time monitoring strategy proposed by Phillips, Shi and Yu
(2015a,b;PSY), along with a new bootstrap procedure designed to mitigate
the potential impact of heteroskedasticity and to effect family-wise
size control in recursive testing algorithms (Phillips and Shi,
forthcoming). This methodology has been shown effective for bubble and
crisis detection (PSY, 2015a,b; Phillips and Shi, 2017) and is now
widely used by academic researchers, central bank economists, and fiscal
regulators.

## Installation

You can install the **stable** version from
[CRAN](https://cran.r-project.org/web/packages/psymonitor/index.html)

``` r
install.packages("psymonitor")
```

You can install the **development** version from
[GitHub](https://github.com/itamarcaspi/psymonitor/)

``` r
# install.packages("devtools")
devtools::install_github("itamarcaspi/psymonitor")
```

## Usage

For the illustration purposes we will use data on the credit risk in the
European sovereign sector, that is proxied by an index constructed as a
GDP weighted 10-year government bond yield of the GIIPS (Greece,
Ireland, Italy, Portugal, and Spain) countries, and comes with the
‘psymonitor’ package.

Let’s walk through some basics. First load the `psymonitor` package and
get data on GIIPS spread.

``` r
library(psymonitor)
data(spread)
```

Next, define a few parameters for the test and the simulation.

``` r
y        <- spread$value
obs      <- length(y)
swindow0 <- floor(obs * (0.01 + 1.8 / sqrt(obs))) # set minimal window size
IC       <- 2  # use BIC to select the number of lags
adflag   <- 6  # set the maximum nuber of lags to 6
yr       <- 2  
Tb       <- 12*yr + swindow0 - 1  # Set the control sample size
nboot    <- 99  # set the number of replications for the bootstrap
```

Next, estimate the PSY test statistic using `PSY()` and its
corresponding bootstrap-based critical values using `cvPSYwmboot()`.

``` r
bsadf          <- PSY(y, swindow0 = swindow0, IC = IC,
                      adflag = adflag)  # estimate the PSY test statistics sequence

quantilesBsadf <- cvPSYwmboot(y, swindow0 = swindow0, IC = IC,
                              adflag = adflag, Tb = Tb, nboot = 99,
                              nCores = 2) # simulate critical values via wild bootstrap. Note that the number of cores is arbitrarily set to 2.
```

Next, identify crisis periods, defined as periods where the test
statistic is above its corresponding critical value, using the
`locate()` function.

``` r
dim          <- obs - swindow0 + 1 
monitorDates <- spread$date[swindow0:obs]
quantile95   <- quantilesBsadf %*% matrix(1, nrow = 1, ncol = dim)
ind95        <- (bsadf > t(quantile95[2, ])) * 1
periods      <- locate(ind95, monitorDates)  # Locate crisis periods
```

Finally, print a table that holds the identified crisis periods with the
help of the `disp()`
function.

``` r
crisisDates <- disp(periods, obs)  #generate table that holds crisis periods
print(crisisDates)
```

|   |      start |        end |
| -: | ---------: | ---------: |
| 1 | 2008-03-01 | 2008-03-01 |
| 2 | 2008-09-01 | 2009-04-01 |
| 3 | 2010-05-01 | 2012-08-01 |

-----

Pleas check the packages’ articles for an elaborated analysis of the
[spreads
data](https://itamarcaspi.github.io/psymonitor/articles/illustrationBONDS.html),
as well as a demonstration using data on the [S\&P 500 price-to-dividend
ratio](https://itamarcaspi.github.io/psymonitor/articles/illustrationSNP.html).

-----

## References

  - Phillips, P. C. B., & Shi, S.(2017). Detecting financial collapse
    and ballooning sovereign risk. Cowles Foundation Discussion Paper
    No. 2110.
  - Phillips, P. C. B., & Shi, S.(forthcoming). Real time monitoring of
    asset markets: Bubbles and crisis. In Hrishikesh D. Vinod and C.R.
    Rao (Eds.), *Handbook of Statistics Volume 41 - Econometrics Using
    R*.
  - Phillips, P. C. B., Shi, S., & Yu, J. (2015a). Testing for multiple
    bubbles: Historical episodes of exuberance and collapse in the S\&P
    500. *International Economic Review*, 56(4), 1034–1078.
  - Phillips, P. C. B., Shi, S., & Yu, J. (2015b). Testing for multiple
    bubbles: Limit Theory for Real-Time Detectors. *International
    Economic Review*, 56(4), 1079–1134.
