<!--
{% comment %}
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to you under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
{% endcomment %}
-->

## Table of Contents

  * [Introduction](#introduction)
  * [Built-In Construction Functions](#built-in-construction-functions)
    * [`tensor`-Function](#tensor-function)
  * [DML-Bodied Built-In functions](#dml-bodied-built-in-functions)
    * [`confusionMatrix`-Function](#confusionmatrix-function)
    * [`cvlm`-Function](#cvlm-function)
    * [`discoverFD`-Function](#discoverFD-function)
    * [`glm`-Function](#glm-function)
    * [`gridSearch`-Function](#gridSearch-function)
    * [`img_brightness`-Function](#img_brightness-function)
    * [`img_crop`-Function](#img_crop-function)
    * [`img_mirror`-Function](#img_mirror-function)
    * [`imputeByFD`-Function](#imputeByFD-function)
    * [`intersect`-Function](#intersect-function)
    * [`KMeans`-Function](#KMeans-function)
    * [`lm`-Function](#lm-function)
    * [`lmDS`-Function](#lmds-function)
    * [`lmCG`-Function](#lmcg-function)
    * [`lmpredict`-Function](#lmpredict-function)
    * [`mice`-Function](#mice-function)
    * [`multiLogReg`-Function](#multiLogReg-function)
    * [`pnmf`-Function](#pnmf-function)
    * [`scale`-Function](#scale-function)
    * [`sigmoid`-Function](#sigmoid-function)
    * [`steplm`-Function](#steplm-function)
    * [`slicefinder`-Function](#slicefinder-function)
    * [`normalize`-Function](#normalize-function)
    * [`gnmf`-Function](#gnmf-function)
    * [`msvm`-Function](#msvm-function)
    * [`naivebayes`-Function](#naivebayes-function)
    * [`outlier`-Function](#outlier-function)
    * [`toOneHot`-Function](#toOneHOt-function)
    * [`winsorize`-Function](#winsorize-function)
    
    
# Introduction

The DML (Declarative Machine Learning) language has built-in functions which enable access to both low- and high-level functions
to support all kinds of use cases.

Builtins are either implemented on a compiler level or as DML scripts that are loaded at compile time.

# Built-In Construction Functions

There are some functions which generate an object for us. They create matrices, tensors, lists and other non-primitive
objects.

## `tensor`-Function

The `tensor`-function creates a **tensor** for us.

### Usage
```r
tensor(data, dims, byRow = TRUE)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| data    | Matrix[?], Tensor[?], Scalar[?] | required | The data with which the tensor should be filled. See [`data`-Argument](#data-argument).|
| dims    | Matrix[Integer], Tensor[Integer], Scalar[String], List[Integer] | required | The dimensions of the tensor. See [`dims`-Argument](#dims-argument). |
| byRow   | Boolean        | TRUE     | NOT USED. Will probably be removed or replaced. |

Note that this function is highly **unstable** and will be overworked and might change signature and functionality.

### Returns
| Type           | Description |
| :------------- | :---------- |
| Tensor[?] | The generated Tensor. Will support more datatypes than `Double`. |

##### `data`-Argument

The `data`-argument can be a `Matrix` of any datatype from which the elements will be taken and placed in the tensor 
until filled. If given as a `Tensor` the same procedure takes place. We iterate through `Matrix` and `Tensor` by starting
with each dimension index at `0` and then incrementing the lowest one, until we made a complete pass over the dimension,
and then increasing the dimension index above. This will be done until the `Tensor` is completely filled.

If `data` is a `Scalar`, we fill the whole tensor with the value.

##### `dims`-Argument

The dimension of the tensor can either be given by a vector represented by either by a `Matrix`, `Tensor`, `String` or `List`.
Dimensions given by a `String` will be expected to be concatenated by spaces.

### Example
```r
print("Dimension matrix:");
d = matrix("2 3 4", 1, 3);
print(toString(d, decimal=1))

print("Tensor A: Fillvalue=3, dims=2 3 4");
A = tensor(3, d); # fill with value, dimensions given by matrix
print(toString(A))

print("Tensor B: Reshape A, dims=4 2 3");
B = tensor(A, "4 2 3"); # reshape tensor, dimensions given by string
print(toString(B))

print("Tensor C: Reshape dimension matrix, dims=1 3");
C = tensor(d, list(1, 3)); # values given by matrix, dimensions given by list
print(toString(C, decimal=1))

print("Tensor D: Values=tst, dims=Tensor C");
D = tensor("tst", C); # fill with string, dimensions given by tensor
print(toString(D))
```

Note that reshape construction is not yet supported for **SPARK** execution.

# DML-Bodied Built-In Functions

**DML-bodied built-in functions** are written as DML-Scripts and executed as such when called.

## `confusionMatrix`-Function

A `confusionMatrix`-accepts a vector for prediction and a one-hot-encoded matrix, then it computes the max value
of each vector and compare them, after which it calculates and returns the sum of classifications and the average of
each true class.

### Usage
```r
confusionMatrix(P, Y)
```

### Arguments

| Name | Type           | Default | Description |
| :--- | :------------- | :------ | :---------- |
| P    | Matrix[Double] | ---     | vector of prediction |
| Y    | Matrix[Double] | ---     | vector of Golden standard One Hot Encoded |

### Returns
 
| Name         | Type           | Description |
| :----------- | :------------- | :---------- |
| ConfusionSum | Matrix[Double] | The Confusion Matrix Sums of classifications |
| ConfusionAvg | Matrix[Double] | The Confusion Matrix averages of each true class |

### Example
 
```r
numClasses = 1
z = rand(rows = 5, cols = 1, min = 1, max = 9)
X = round(rand(rows = 5, cols = 1, min = 1, max = numClasses))
y = toOneHot(X, numClasses)
[ConfusionSum, ConfusionAvg] = confusionMatrix(P=z, Y=y)
```

## `cvlm`-Function

The `cvlm`-function is used for cross-validation of the provided data model. This function follows a non-exhaustive
cross validation method. It uses [`lm`](#lm-function) and [`lmpredict`](#lmpredict-function) functions to solve the linear
regression and to predict the class of a feature vector with no intercept, shifting, and rescaling.

### Usage
```r
cvlm(X, y, k)
```

### Arguments
| Name | Type           | Default  | Description |
| :--- | :------------- | :------- | :---------- |
| X    | Matrix[Double] | required | Recorded Data set into matrix |
| y    | Matrix[Double] | required | 1-column matrix of response values.  |
| k    | Integer        | required | Number of subsets needed, It should always be more than `1` and less than `nrow(X)` |
| icpt | Integer        | `0`      | Intercept presence, shifting and rescaling the columns of X |
| reg  | Double         | `1e-7`   | Regularization constant (lambda) for L2-regularization. set to nonzero for highly dependant/sparse/numerous features |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | Response values |
| Matrix[Double] | Validated data set |

### Example
```r
X = rand (rows = 5, cols = 5)
y = X %*% rand(rows = ncol(X), cols = 1)
[predict, beta] = cvlm(X = X, y = y, k = 4)
```

## `discoverFD`-Function

The `discoverFD`-function finds the functional dependencies.

### Usage
```r
discoverFD(X, Mask, threshold)
```

### Arguments
| Name      | Type   | Default | Description |
| :-------- | :----- | ------- | :---------- |
| X         | Double | --      | Input Matrix X, encoded Matrix if data is categorical |
| Mask      | Double | --      | A row vector for interested features i.e. Mask =[1, 0, 1] will exclude the second column from processing |
| threshold | Double | --      | threshold value in interval [0, 1] for robust FDs |

### Returns
| Type   | Description |
| :----- | :---------- |
| Double | matrix of functional dependencies |


## `glm`-Function

The `glm`-function  is a flexible generalization of ordinary linear regression that allows for response variables that have
error distribution models.

### Usage
```r
glm(X,Y)
```

### Arguments
| Name | Type           | Default  | Description |
| :--- | :------------- | :------- | :---------- |
| X    | Matrix[Double] | required | matrix X of feature vectors |
| Y    | Matrix[Double] | required | matrix Y with either 1 or 2 columns: if dfam = 2, Y is 1-column Bernoulli or 2-column Binomial (#pos, #neg) |
| dfam | Int            | `1`      | Distribution family code: 1 = Power, 2 = Binomial |
| vpow | Double         | `0.0`    | Power for Variance defined as (mean)^power (ignored if dfam != 1):  0.0 = Gaussian, 1.0 = Poisson, 2.0 = Gamma, 3.0 = Inverse Gaussian |
| link | Int            | `0`      | Link function code: 0 = canonical (depends on distribution), 1 = Power, 2 = Logit, 3 = Probit, 4 = Cloglog, 5 = Cauchit |
| lpow | Double         | `1.0`    | Power for Link function defined as (mean)^power (ignored if link != 1):  -2.0 = 1/mu^2, -1.0 = reciprocal, 0.0 = log, 0.5 = sqrt, 1.0 = identity |
| yneg | Double         | `0.0`    | Response value for Bernoulli "No" label, usually 0.0 or -1.0 |
| icpt | Int            | `0`      | Intercept presence, X columns shifting and rescaling: 0 = no intercept, no shifting, no rescaling; 1 = add intercept, but neither shift nor rescale X; 2 = add intercept, shift & rescale X columns to mean = 0, variance = 1 |
| reg  | Double         | `0.0`    | Regularization parameter (lambda) for L2 regularization |
| tol  | Double         | `1e-6`   | Tolerance (epislon) value. |
| disp | Double         | `0.0`    | (Over-)dispersion value, or 0.0 to estimate it from data |
| moi  | Int            | `200`    | Maximum number of outer (Newton / Fisher Scoring) iterations |
| mii  | Int            | `0`      | Maximum number of inner (Conjugate Gradient) iterations, 0 = no maximum |

### Returns
| Type           | Description      |
| :------------- | :--------------- |
| Matrix[Double] | Matrix whose size depends on icpt ( icpt=0: ncol(X) x 1;  icpt=1: (ncol(X) + 1) x 1;  icpt=2: (ncol(X) + 1) x 2) |

### Example
```r
X = rand (rows = 5, cols = 5 )
y = X %*% rand(rows = ncol(X), cols = 1)
beta = glm(X=X,Y=y)
```

## `gridSearch`-Function

The `gridSearch`-function is used to find the optimal hyper-parameters of a model which results in the most _accurate_
predictions. This function takes `train` and `eval` functions by name. 

### Usage
```r
gridSearch(X, y, train, predict, params, paramValues, verbose)
```

### Arguments
| Name         | Type           | Default  | Description |
| :------      | :------------- | -------- | :---------- |
| X            | Matrix[Double] | required | Input Matrix of vectors. |
| y            | Matrix[Double] | required | Input Matrix of vectors. |
| train        | String         | required | Specified training function. |
| predict      | String         | required | Evaluation based function. |
| params       | List[String]   | required | List of parameters |
| paramValues  | List[Unknown]  | required | Range of values for the parameters |
| verbose      | Boolean        | `TRUE`   | If `TRUE` print messages are activated |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | Parameter combination |
| Frame[Unknown] | Best results model |

### Example
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
params = list("reg", "tol", "maxi")
paramRanges = list(10^seq(0,-4), 10^seq(-5,-9), 10^seq(1,3))
[B, opt]= gridSearch(X=X, y=y, train="lm", predict="lmPredict", params=params, paramValues=paramRanges, verbose = TRUE)
```

## `img_brightness`-Function

The `img_brightness`-function is an image data augumentation function.
It changes the brightness of the image.

### Usage
```r
img_brightness(img_in, value, channel_max)
```

### Arguments
| Name        | Type           | Default  | Description |
| :---------- | :------------- | -------- | :---------- |
| img_in      | Matrix[Double] | ---      | Input matrix/image |
| value       | Double         | ---      | The amount of brightness to be changed for the image |
| channel_max | Integer        | ---      | Maximum value of the brightness of the image |

### Returns
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| img_out | Matrix[Double] | ---      | Output matrix/image |

### Example
```r
A = rand(rows = 3, cols = 3, min = 0, max = 255)
B = img_brightness(img_in = A, value = 128, channel_max = 255)
```

## `img_crop`-Function

The `img_crop`-function is an image data augumentation function.
It cuts out a subregion of an image.

### Usage
```r
img_crop(img_in, w, h, x_offset, y_offset)
```

### Arguments
| Name     | Type           | Default  | Description |
| :------  | :------------- | -------- | :---------- |
| img_in   | Matrix[Double] | ---      | Input matrix/image |
| w        | Integer        | ---      | The width of the subregion required  |
| h        | Integer        | ---      | The height of the subregion required |
| x_offset | Integer        | ---      | The horizontal coordinate in the image to begin the crop operation |
| y_offset | Integer        | ---      | The vertical coordinate in the image to begin the crop operation |

### Returns
| Name    | Type           | Default | Description |
| :------ | :------------- | ------- | :---------- |
| img_out | Matrix[Double] | ---     | Cropped matrix/image |

### Example
```r
A = rand(rows = 3, cols = 3, min = 0, max = 255) 
B = img_crop(img_in = A, w = 20, h = 10, x_offset = 0, y_offset = 0)
```

## `img_mirror`-Function

The `img_mirror`-function is an image data augumentation function.
It flips an image on the `X` (horizontal) or `Y` (vertical) axis. 

### Usage
```r
img_mirror(img_in, horizontal_axis)
```

### Arguments
| Name            | Type           | Default  | Description |
| :-------------- | :------------- | -------- | :---------- |
| img_in          | Matrix[Double] | ---      | Input matrix/image |
| horizontal_axis | Boolean        | ---      | If TRUE, the  image is flipped with respect to horizontal axis otherwise vertical axis |

### Returns
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| img_out | Matrix[Double] | ---      | Flipped matrix/image |

### Example
```r
A = rand(rows = 3, cols = 3, min = 0, max = 255)
B = img_mirror(img_in = A, horizontal_axis = TRUE)
```

## `imputeByFD`-Function

The `imputeByFD`-function imputes missing values from observed values (if exist)
using robust functional dependencies.

### Usage
```r
imputeByFD(F, sourceAttribute, targetAttribute, threshold)
```

### Arguments
| Name      | Type    | Default  | Description |
| :-------- | :------ | -------- | :---------- |
| F         | String  | --       | A data frame |
| source    | Integer | --       | Source attribute to use for imputation and error correction |
| target    | Integer | --       | Attribute to be fixed |
| threshold | Double  | --       | threshold value in interval [0, 1] for robust FDs |

### Returns
| Type   | Description |
| :----- | :---------- |
| String | Frame with possible imputations |


## `KMeans`-Function

The kmeans() implements the KMeans Clustering algorithm.

### Usage
```r
kmeans(X = X, k = 20, runs = 10, max_iter = 5000, eps = 0.000001, is_verbose = FALSE, avg_sample_size_per_centroid = 50)
```

### Arguments
| Name       | Type            | Default    | Description |
| :--------- | :-------------- | :--------- | :---------- |
| x          | Matrix[Double]  | required   | The input Matrix to do KMeans on. |
| k          | Int             | `10`       | Number of centroids |
| runs       | Int             | `10`       | Number of runs (with different initial centroids) |
| max_iter   | Int             | `100`      |Max no. of iterations allowed |
| eps        | Double          | `0.000001` | Tolerance (epsilon) for WCSS change ratio |
| is_verbose | Boolean         |   FALSE    | do not print per-iteration stats |

### Returns
| Type   | Description |
| :----- | :---------- |
| String | The mapping of records to centroids |
| String | The output matrix with the centroids |


## `lm`-Function

The `lm`-function solves linear regression using either the **direct solve method** or the **conjugate gradient algorithm**
depending on the input size of the matrices (See [`lmDS`-function](#lmds-function) and 
[`lmCG`-function](#lmcg-function) respectively).

### Usage
```r
lm(X, y, icpt = 0, reg = 1e-7, tol = 1e-7, maxi = 0, verbose = TRUE)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| y       | Matrix[Double] | required | 1-column matrix of response values. |
| icpt    | Integer        | `0`      | Intercept presence, shifting and rescaling the columns of X ([Details](#icpt-argument))|
| reg     | Double         | `1e-7`   | Regularization constant (lambda) for L2-regularization. set to nonzero for highly dependant/sparse/numerous features|
| tol     | Double         | `1e-7`   | Tolerance (epsilon); conjugate gradient procedure terminates early if L2 norm of the beta-residual is less than tolerance * its initial norm|
| maxi    | Integer        | `0`      | Maximum number of conjugate gradient iterations. 0 = no maximum |
| verbose | Boolean        | `TRUE`   | If `TRUE` print messages are activated |

Note that if number of *features* is small enough (`rows of X/y < 2000`), the [`lmDS`-Function'](#lmds-function)
is called internally and parameters `tol` and `maxi` are ignored.

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of weights. |

##### `icpt`-Argument

The *icpt-argument* can be set to 3 modes:
 
  * 0 = no intercept, no shifting, no rescaling
  * 1 = add intercept, but neither shift nor rescale X
  * 2 = add intercept, shift & rescale X columns to mean = 0, variance = 1

### Example
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
lm(X = X, y = y)
```

## `intersect`-Function

The `intersect`-function implements set intersection for numeric data.

### Usage
```r
intersect(X, Y)
```

### Arguments
| Name | Type   | Default  | Description |
| :--- | :----- | -------- | :---------- |
| X    | Double | --       | matrix X, set A |
| Y    | Double | --       | matrix Y, set B | 

### Returns
| Type   | Description |
| :----- | :---------- |
| Double | intersection matrix, set of intersecting items |


## `lmDS`-Function

The `lmDS`-function solves linear regression by directly solving the *linear system*.

### Usage
```r
lmDS(X, y, icpt = 0, reg = 1e-7, verbose = TRUE)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| y       | Matrix[Double] | required | 1-column matrix of response values. |
| icpt    | Integer        | `0`      | Intercept presence, shifting and rescaling the columns of X ([Details](#icpt-argument))|
| reg     | Double         | `1e-7`   | Regularization constant (lambda) for L2-regularization. set to nonzero for highly dependant/sparse/numerous features|
| verbose | Boolean        | `TRUE`   | If `TRUE` print messages are activated |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of weights. |

### Example
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
lmDS(X = X, y = y)
```

## `lmCG`-Function

The `lmCG`-function solves linear regression using the *conjugate gradient algorithm*.

### Usage
```r
lmCG(X, y, icpt = 0, reg = 1e-7, tol = 1e-7, maxi = 0, verbose = TRUE)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| y       | Matrix[Double] | required | 1-column matrix of response values. |
| icpt    | Integer        | `0`      | Intercept presence, shifting and rescaling the columns of X ([Details](#icpt-argument))|
| reg     | Double         | `1e-7`   | Regularization constant (lambda) for L2-regularization. set to nonzero for highly dependant/sparse/numerous features|
| tol     | Double         | `1e-7`   | Tolerance (epsilon); conjugate gradient procedure terminates early if L2 norm of the beta-residual is less than tolerance * its initial norm|
| maxi    | Integer        | `0`      | Maximum number of conjugate gradient iterations. 0 = no maximum |
| verbose | Boolean        | `TRUE`   | If `TRUE` print messages are activated |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of weights. |

### Example
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
lmCG(X = X, y = y, maxi = 10)
```

## `lmpredict`-Function

The `lmpredict`-function predicts the class of a feature vector.

### Usage
```r
lmpredict(X, w)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vector(s). |
| w       | Matrix[Double] | required | 1-column matrix of weights. |
| icpt    | Matrix[Double] | `0`      | Intercept presence, shifting and rescaling of X ([Details](#icpt-argument))|

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of classes. |

### Example
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
w = lm(X = X, y = y)
yp = lmpredict(X, w)
```

## `mice`-Function

The `mice`-function implements Multiple Imputation using Chained Equations (MICE) for nominal data.

### Usage
```r
mice(F, cMask, iter, complete, verbose)
```

### Arguments
| Name     | Type           | Default  | Description |
| :------- | :------------- | -------- | :---------- |
| F        | Frame[String]  | required | Data Frame with one-dimensional row matrix with N columns where N>1. |
| cMask    | Matrix[Double] | required | 0/1 row vector for identifying numeric (0) and categorical features (1) with one-dimensional row matrix with column = ncol(F). |
| iter     | Integer        | `3`      | Number of iteration for multiple imputations. |
| complete | Integer        | `3`      | A complete dataset generated though a specific iteration. |
| verbose  | Boolean        | `FALSE`  | Boolean value. |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Frame[String]  | imputed dataset. |
| Frame[String]  | A complete dataset generated though a specific iteration. |

### Example
```r
F = as.frame(matrix("4 3 2 8 7 8 5", rows=1, cols=7))
cMask = round(rand(rows=1,cols=ncol(F),min=0,max=1))
[dataset, singleSet] = mice(F, cMask, iter = 3, complete = 3, verbose = FALSE)
```

## `multiLogReg`-Function

The `multiLogReg`-function solves Multinomial Logistic Regression using Trust Region method.
(See: Trust Region Newton Method for Logistic Regression, Lin, Weng and Keerthi, JMLR 9 (2008) 627-650)

### Usage
```r
multiLogReg(X, Y, icpt, reg, tol, maxi, maxii, verbose)
```

### Arguments
| Name  | Type   | Default | Description |
| :---- | :----- | ------- | :---------- |
| X     | Double | --      | The matrix of feature vectors |
| Y     | Double | --      | The matrix with category labels |
| icpt  | Int    | `0`     | Intercept presence, shifting and rescaling X columns: 0 = no intercept, no shifting, no rescaling; 1 = add intercept, but neither shift nor rescale X; 2 = add intercept, shift & rescale X columns to mean = 0, variance = 1 |
| reg   | Double | `0`     | regularization parameter (lambda = 1/C); intercept is not regularized |
| tol   | Double | `1e-6`  | tolerance ("epsilon") |
| maxi  | Int    | `100`   | max. number of outer newton interations |
| maxii | Int    | `0`     | max. number of inner (conjugate gradient) iterations |

### Returns
| Type   | Description |
| :----- | :---------- |
| Double | Regression betas as output for prediction |

### Example
```r
X = rand(rows = 50, cols = 30)
Y = X %*% rand(rows = ncol(X), cols = 1)
betas = multiLogReg(X = X, Y = Y, icpt = 2,  tol = 0.000001, reg = 1.0, maxi = 100, maxii = 20, verbose = TRUE)
```


## `pnmf`-Function

The `pnmf`-function implements Poisson Non-negative Matrix Factorization (PNMF). Matrix `X` is factorized into
two non-negative matrices, `W` and `H` based on Poisson probabilistic assumption. This non-negativity makes the
resulting matrices easier to inspect.

### Usage
```r
pnmf(X, rnk, eps = 10^-8, maxi = 10, verbose = TRUE)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| rnk     | Integer        | required | Number of components into which matrix X is to be factored. |
| eps     | Double         | `10^-8`  | Tolerance |
| maxi    | Integer        | `10`     | Maximum number of conjugate gradient iterations. |
| verbose | Boolean        | TRUE     | If TRUE, 'iter' and 'obj' are printed.|


### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | List of pattern matrices, one for each repetition. |
| Matrix[Double] | List of amplitude matrices, one for each repetition. |

### Example
```r
X = rand(rows = 50, cols = 10)
[W, H] = pnmf(X = X, rnk = 2, eps = 10^-8, maxi = 10, verbose = TRUE)
```

## `scale`-Function

The scale function is a generic function whose default method centers or scales the column of a numeric matrix.

### Usage
```r
scale(X, center=TRUE, scale=TRUE)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| center  | Boolean        | required | either a logical value or numerical value. |
| scale   | Boolean        | required | either a logical value or numerical value. |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of weights. |

### Example
```r
X = rand(rows = 20, cols = 10)
center=TRUE;
scale=TRUE;
Y= scale(X,center,scale)
```

## `sigmoid`-Function

The Sigmoid function is a type of activation function, and also defined as a squashing function which limit the output 
to a range between 0 and 1, which will make these functions useful in the prediction of probabilities.

### Usage
```r
sigmoid(X)
```

### Arguments
| Name  | Type           | Default  | Description |
| :---- | :------------- | -------- | :---------- |
| X     | Matrix[Double] | required | Matrix of feature vectors. |


### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of weights. |

### Example
```r
X = rand (rows = 20, cols = 10)
Y = sigmoid(X)
```

## `steplm`-Function

The `steplm`-function (stepwise linear regression) implements a classical forward feature selection method.
This method iteratively runs what-if scenarios and greedily selects the next best feature until the Akaike 
information criterion (AIC) does not improve anymore. Each configuration trains a regression model via `lm`,
which in turn calls either the closed form `lmDS` or iterative `lmGC`.

### Usage
```r
steplm(X, y, icpt);
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| y       | Matrix[Double] | required | 1-column matrix of response values. |
| icpt    | Integer        | `0`      | Intercept presence, shifting and rescaling the columns of X ([Details](#icpt-argument))|
| reg     | Double         | `1e-7`   | Regularization constant (lambda) for L2-regularization. set to nonzero for highly dependent/sparse/numerous features|
| tol     | Double         | `1e-7`   | Tolerance (epsilon); conjugate gradient procedure terminates early if L2 norm of the beta-residual is less than tolerance * its initial norm|
| maxi    | Integer        | `0`      | Maximum number of conjugate gradient iterations. 0 = no maximum |
| verbose | Boolean        | `TRUE`   | If `TRUE` print messages are activated |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | Matrix of regression parameters (the betas) and its size depend on `icpt` input value. (C in the example)|
| Matrix[Double] | Matrix of `selected` features ordered as computed by the algorithm. (S in the example)|

##### `icpt`-Argument

The *icpt-arg* can be set to 2 modes:
 
  * 0 = no intercept, no shifting, no rescaling
  * 1 = add intercept, but neither shift nor rescale X

##### `selected`-Output

If the best AIC is achieved without any features the matrix of *selected* features contains 0. Moreover, in this case no further statistics will be produced 

### Example
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
[C, S] = steplm(X = X, y = y, icpt = 1);
```

## `slicefinder`-Function

The `slicefinder`-function returns top-k worst performing subsets according to a model calculation.

### Usage
```r
slicefinder(X,W, y, k, paq, S);
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Recoded dataset into Matrix |
| W       | Matrix[Double] | required | Trained model |
| y       | Matrix[Double] | required | 1-column matrix of response values. |
| k       | Integer        | 1        | Number of subsets required |
| paq     | Integer        | 1        | amount of values wanted for each col, if paq = 1 then its off |
| S       | Integer        | 2        | amount of subsets to combine (for now supported only 1 and 2) |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | Matrix containing the information of top_K slices (relative error, standart error, value0, value1, col_number(sort), rows, cols,range_row,range_cols, value00, value01,col_number2(sort), rows2, cols2,range_row2,range_cols2) |

### Usage
```r
X = rand (rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
w = lm(X = X, y = y)
ress = slicefinder(X = X,W = w, Y = y,  k = 5, paq = 1, S = 2);
```

## `normalize`-Function

The `normalize`-function normalises the values of a matrix by changing the dataset to use a common scale.
This is done while preserving differences in the ranges of values. 
The output is a matrix of values in range [0,1].

### Usage
```r
normalize(X); 
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |


### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | 1-column matrix of normalized values. |



### Example
```r
X = rand(rows = 50, cols = 10)
y = X %*% rand(rows = ncol(X), cols = 1)
y = normalize(X = X)
```

## `gnmf`-Function

The `gnmf`-function does Gaussian Non-Negative Matrix Factorization.
In this, a matrix X is factorized into two matrices W and H, such that all three matrices have no negative elements.
This non-negativity makes the resulting matrices easier to inspect.

### Usage
```r
gnmf(X, rnk, eps = 10^-8, maxi = 10)
```

### Arguments
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| X       | Matrix[Double] | required | Matrix of feature vectors. |
| rnk     | Integer        | required | Number of components into which matrix X is to be factored. |
| eps     | Double         | `10^-8`  | Tolerance |
| maxi    | Integer        | `10`     | Maximum number of conjugate gradient iterations. |


### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | List of pattern matrices, one for each repetition. |
| Matrix[Double] | List of amplitude matrices, one for each repetition. |

### Example
```r
X = rand(rows = 50, cols = 10)
W = rand(rows = nrow(X), cols = 2, min = -0.05, max = 0.05);
H = rand(rows = 2, cols = ncol(X), min = -0.05, max = 0.05);
gnmf(X = X, rnk = 2, eps = 10^-8, maxi = 10)
```

## `naivebayes`-Function

The `naivebayes`-function computes the class conditional probabilities and class priors.

### Usage
```r
naivebayes(D, C, laplace, verbose)
```

### Arguments
| Name            | Type           | Default  | Description |
| :------         | :------------- | -------- | :---------- |
| D               | Matrix[Double] | required | One dimensional column matrix with N rows. |
| C               | Matrix[Double] | required | One dimensional column matrix with N rows. |
| Laplace         | Double         | `1`      | Any Double value. |
| Verbose         | Boolean        | `TRUE`   | Boolean value. |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | Class priors, One dimensional column matrix with N rows. |
| Matrix[Double] | Class conditional probabilites, One dimensional column matrix with N rows. |

### Example
```r
D=rand(rows=10,cols=1,min=10)
C=rand(rows=10,cols=1,min=10)
[prior, classConditionals] = naivebayes(D, C, laplace = 1, verbose = TRUE)
```

## `outlier`-Function

This `outlier`-function takes a matrix data set as input from where it determines which point(s)
have the largest difference from mean.

### Usage
```r
outlier(X, opposite)
```

### Arguments
| Name     | Type           | Default  | Description |
| :------- | :------------- | -------- | :---------- |
| X        | Matrix[Double] | required | Matrix of Recoded dataset for outlier evaluation |
| opposite | Boolean        | required | (1)TRUE for evaluating outlier from upper quartile range, (0)FALSE for evaluating outlier from lower quartile range |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | matrix indicating outlier values |

### Example
```r
X = rand (rows = 50, cols = 10)
outlier(X=X, opposite=1)
```

## `toOneHot`-Function

The `toOneHot`-function encodes unordered categorical vector to multiple binarized vectors.

### Usage
```r
toOneHot(X, numClasses)
```

### Arguments
| Name       | Type           | Default  | Description |
| :--------- | :------------- | -------- | :---------- |
| X          | Matrix[Double] | required | vector with N integer entries between 1 and numClasses. |
| numClasses | int            | required | number of columns, must be greater than or equal to largest value in X. |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | one-hot-encoded matrix with shape (N, numClasses). |

### Example
```r
numClasses = 5
X = round(rand(rows = 10, cols = 10, min = 1, max = numClasses))
y = toOneHot(X,numClasses)
```

## `msvm`-Function

The `msvm`-function implements builtin multiclass SVM with squared slack variables
It learns one-against-the-rest binary-class classifiers by making a function call to l2SVM

### Usage
```r
msvm(X, Y, intercept, epsilon, lamda, maxIterations, verbose)
```


### Arguments
| Name          | Type             | Default    | Description |
| :------       | :-------------   | --------   | :---------- |
| X             | Double           | ---        | Matrix X of feature vectors.|
| Y             | Double           | ---        | Matrix Y of class labels. |
| intercept     | Boolean          | False      | No Intercept ( If set to TRUE then a constant bias column is added to X)|
| num_classes   | Integer          | 10         | Number of classes.|
| epsilon       | Double           | 0.001      | Procedure terminates early if the reduction in objective function value is less than epsilon (tolerance) times the initial objective function value.|
| lamda         | Double           | 1.0        | Regularization parameter (lambda) for L2 regularization|
| maxIterations | Integer          | 100        | Maximum number of conjugate gradient iterations|
| verbose       | Boolean          | False      | Set to true to print while training.|


### Returns
| Name    | Type           | Default  | Description |
| :------ | :------------- | -------- | :---------- |
| model   | Double         | ---      | Model matrix. |


### Example
```r
X = rand(rows = 50, cols = 10)
y = round(X %*% rand(rows=ncol(X), cols=1))
model = msvm(X = X, Y = y, intercept = FALSE, epsilon = 0.005, lambda = 1.0, maxIterations = 100, verbose = FALSE)
```

## `winsorize`-Function

The `winsorize`-function removes outliers from the data. It does so by computing upper and lower quartile range
of the given data then it replaces any value that falls outside this range (less than lower quartile range or more
than upper quartile range).

### Usage
```r
winsorize(X)
```

### Arguments
| Name     | Type           | Default  | Description |
| :------- | :------------- | :--------| :---------- |
| X        | Matrix[Double] | required | recorded data set with possible outlier values |

### Returns
| Type           | Description |
| :------------- | :---------- |
| Matrix[Double] | Matrix without outlier values |

### Example
```r
X = rand(rows=10, cols=10,min = 1, max=9)
Y = winsorize(X=X)
```
