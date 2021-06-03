# ComBat harmonization in R

##### Software status

| Resource:   | Travis CI                                                                                                                                            |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Platform:   | Linux                                                                                                                                                |
| R CMD check | <a href="https://travis-ci.org/Jfortin1/ComBatHarmonization"><img src="https://travis-ci.org/Jfortin1/ComBatHarmonization.svg?branch=master" alt="Build status"></a> |

## Table of content
- [1. Installation](#id-section1)
- [2. Harmonization](#id-section2)
- [3. Visualization](#id-section3)

<div id='id-section1'/>



## 1. Installation
 
neuroCombat can be installed in R by typing the following commands:
```{r}
library(devtools)
install_github("jfortin1/CombatHarmonization/R/neuroCombat")
```

<div id='id-section2'/>

## 2. Multi-Site Harmonization

ComBat estimates scanner-specific location and scale parameters, for each feature separately, and pools information across features using empirical Bayes to improve the estimation of those parameters for small sample size studies.  

### 2.1 Full ComBat with empirical Bayes

The  `neuroCombat` function is the main function. It requires two mandatory arguments:
- a data matrix (p x n) `dat` for which the p rows are features, and the n columns are participants. 
- a numeric or character vector `batch` of length n indicating the site/scanner/study id. 

For illustration purpose, let's simulate an imaging dataset with n=10 participants, acquired on 2 scanners, with 5 participants each, with p=10000 voxels per scan. 

```r
library(neuroCombat)
p=10000
n=10
batch = c(1,1,1,1,1,2,2,2,2,2) #Batch variable for the scanner id
dat = matrix(runif(p*n), p, n) #Random Data matrix
```
We use the function `neuroCombat` to harmonize the data across the 2 scanners:

```r
data.harmonized <- neuroCombat(dat=dat, batch=batch)
```
By default, this uses parametric adjustments. To following command must be used for non-parametric adjustments:
```r
data.harmonized <- neuroCombat(dat=dat, batch=batch, parametric=FALSE)
```

The harmonized matrix is stored in `data.harmonized$dat.combat`. The `data.harmonized` object also contains the different parameters estimated by ComBat:
- `gamma.hat` and `delta.hat`: Estimated location and shift (L/S) parameters before empirical Bayes.
- `gamma.star` and `delta.star`: Empirical Bayes estimated L/S parameters.
- `gamma.bar`, `t2`, `a.prior` and `b.prior`: esimated prior distributions parameters.

`neuroCombat` also accepts an optional argument, `mod`, which is a matrix containing biological covariates, including the outcome of interest. This is recommended to ensure that biological variability is preserved in the harmonization process. For instance, for a study with age and disease covariates,
```r
age <- c(82,70,68,66,80,69,72,76,74,80) # Continuous variable
disease <- as.factor(c(1,2,1,2,1,2,1,2,1,2)) # Categorical variable
```
we first create a model matrix for these two biological covariates using the `model.matrix` function:
```r
mod <- model.matrix(~age+disease)
mod
> mod
   (Intercept) age disease2
1            1  82        0
2            1  70        1
3            1  68        0
4            1  66        1
5            1  80        0
6            1  69        1
7            1  72        0
8            1  76        1
9            1  74        0
10           1  80        1
```
The matrix `mod` is a n x 3 matrix, containing an intercept, age and a dummy variable for the second level of the disease variable (the first level is taken as the baseline group). Note that including an intercept in the model matrix will not change the results of the algorithm; ComBat automatically removes the intercept from the model matrix when fitting the models. We now harmonize the data:

```r
combat.harmonized <- neuroCombat(dat=dat, batch=batch, mod=mod)
```

### 2.2 ComBat without empirical Bayes

Sometimes, it is preferable not to pool information across features, for instance if:
- (1) The number of features is substantially smaller than the number of participants (p << n) or
- (2) The prior distributions used in ComBat do not fit well the data
- (3) The site effects are only present for a small subset of features

An example of (2) is studies with site/scanner effects that are highly heteregenous across features, for instance differential scanner effects between white matter (WM) or grey matter (GM) voxels exist. To run the ComBat model without empirical Bayes, which boils down to fitting a location/shift (L/S) model for each feature separately, the option `eb=FALSE` can be used:

```r
data.harmonized <- neuroCombat(dat=dat, batch=batch, eb=FALSE)
```


### 2.3 ComBat with mean site effects adjustment only (no variance adjustment) 



<div id='id-section3'/>

## 3. Visualization

Coming soon.



