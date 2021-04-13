---
title: Creating more robust functions in R
subtitle: Fail early.

# Summary for listings and search engines
summary: Robust functions in R through testing function inputs and outputs.

# Link this post with a project
projects: []

# Date published
date: "2021-04-06T00:00:00Z"

# Date updated
lastmod: "2021-04-13T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
  focal_point: ""
  placement: 2
  preview_only: false

authors:
- michiel-van-de-ven

tags:
- R
- testing
- debugging

categories:
- R
- Testing
- Debugging
---
In the code below, I demonstrate how easy it can be to improve the robustness
of custom functions in R. First, I'll show how the function inputs can be 
checked for validity and how including error and warnings into the function 
can lead to easier debugging of faulty code. Second, we will move on to checking the output of functions,
by introducing the `testthat` package and writing a simple unit test. 

In this example, we will use a very simple function without error handling and without checking
the function inputs and outputs. The function divides two vectors of numbers
with length greater than zero.

```r {linenos=table,linenostart=1}
func_divider <- function(numerator, denominator) {
  
  fraction <- numerator/denominator
  return(fraction)
}
```
## Validity of function inputs
What if we want to protect ourselves from supplying this function with invalid
inputs? There a number of situations we want to guard against, for instance:

* Unequal lengths of both the `numerator` and `denominator` vectors;
* An empty `denominator` vector or both input vectors empty;
* Calculate using `NA` values by providing the option to remove these;
* Incorrect data types.

The `stopifnot()` function from base R provides functionality to evaluate function
inputs and is a very useful first step to protect ourselves against illegal function
inputs. `stopifnot()` also tells you why a function input was not valid, i.e.,
which specified condition was not satisfied.
```r {linenos=table,linenostart=1}
func_divider <- function(numerator, denominator, na.rm = FALSE) {
  
  stopifnot(
            # Are numerator and denominator of equal length?
            length(numerator) == length(denominator),
            # Are numerator and denominator both numeric?
            is.numeric(numerator) & is.numeric(denominator),
            # Is the length of numerator and denominator greater than zero?
            length(numerator) > 0 & length(denominator) > 0,
            # Is input for na.rm a Boolean and length 1?
            is.logical(na.rm), length(na.rm) == 1
            )

  # Remove NA's from both input vectors when na.rm is set to TRUE.
  if (na.rm == TRUE) {
    miss <- is.na(numerator) | is.na(denominator)
    numerator <- numerator[!miss]
    denominator <- denominator[!miss]
  }
  
  fraction <- numerator/denominator
  return(fraction)
}
```
`stopifnot()` also tells you why a function input was not valid by prints the
corresponding error message to the console:
```r {linenos=table,linenostart=1}
func_divider(1:3, 1:2)
```
```
Error in func_divider(1:3, 1:2) : 
  length(numerator) == length(denominator) is not TRUE
```
There may also be cases when it is useful to let the function continue its
execution and just produce a warning message. In our example function, it could
be useful to print an warning message when the `denominator` vector contains a
zero In the code below, the function is updated to print a warning message in
the console when the value of zero is detected in the denominator.
```r {linenos=table,hl_lines=["15-19"],linenostart=1}
func_divider <- function(numerator, denominator, na.rm = FALSE) {
  
  
  stopifnot(
            # Are numerator and denominator of equal length?
            length(numerator) == length(denominator),
            # Are numerator and denominator both numeric?
            is.numeric(numerator) & is.numeric(denominator),
            # Is the length of numerator and denominator greater than zero?
            length(numerator) > 0 & length(denominator) > 0,
            # Is input for na.rm a Boolean and length 1?
            is.logical(na.rm), length(na.rm) == 1
            )
  
  # Does the denominator vector contain a zero?
  #If so, print an warning message
  if(0 %in% denominator) {
    warning("Dividing by zero is undefined")
  }

  # Remove NA's from both input vectors when na.rm is set to TRUE.
  if (na.rm == TRUE) {
    miss <- is.na(numerator) | is.na(denominator)
    numerator <- numerator[!miss]
    denominator <- denominator[!miss]
  }
  
  fraction <- numerator/denominator
  return(fraction)
}

func_divider(1, 0)
```
```
Dividing by zero is undefined[1] Inf
```

When you divide by zero, R prints `inf` as the output. There are very few cases
in which it is useful or intentional to divide by zero, so you could also swap
out the warning with an error. However, it is not always easy to decide when to
generate warnings or errors. As a rule of thumb, I'd advise to generate errors
when executing the function otherwise would invalidate the output from the
function. In that case, it would be a waste of time to continue execution.

## Function outputs
Function outputs can be checked interactively by running the below in the R
console, but in my opinion, this is only useful when writing and testing the
function initially. 

```r {linenos=table,linenostart=1}
func_divider(2, 4)
```
```
[1] 0.5
```
```r {linenos=table,linenostart=1}
func_divider(0, 0)
```
```
[1] NaN
Warning message:
In func_divider(0, 0) : Dividing by zero is undefined
```
However, this is not very efficient as testing all combinations of 
input parameters would be very time consuming. A more structured approach to
testing pieces of code is unit testing, in which a test is created for a piece of code (i.e., a unit).
The unit test should tell the user whether the code is still fit for use. 
Otherwise, an error should be produced and execution stopped.

The package `testthat` provides a set of functions to check whether the result 
of the function is as expected. Typically, an R script contains multiple custom
functions that should be tested, and thus, multiple unit tests are required.

You want to keep the unit tests themselves in separate R scripts, and it is good practice
to organize these unit tests in a separate folder. I put mine in folder `tests`.

`testthat` provides a number of different ways to test the function output. For instance,
you can directly evaluate the value of the output: Does the function produce a
three when I expect a three? Similarly to checking the function inputs, you can evaluate 
the length of the output. The package provides a large variety of different 
functions to use in your unit test. All of them are listed in the [package documentation](https://testthat.r-lib.org/reference/index.html).

A unit test for our example function could be something like this:
```r {linenos=table,linenostart=1}
# Load script that contains the unit to be tested
#source("path/to/the/code/that/needs/to/be/tested.R")

# Load libraries
library(testthat)

test_divider <- function() {
  
  # I expect output of 2 when I divide 4 by 2
  expect_equal(func_divider(4, 2), 2)
  
  # I expect an Inf value when I divide by zero
  expect_equal(func_divider(4, 0), Inf) # 4/0 == Inf
  
  # I also expect a warning when I divide by zero
  expect_warning(func_divider(4, 0))
  
  # I expect that the function output is of type 'double' 
  expect_type(func_divider(4, 2), "double")
  
  # I expect that 4/2 is greater than 2/2
  expect_gt(func_divider(4, 2), func_divider(2, 2))
  
  # I expect an error when I input multiple values for argument 'na.rm'
  expect_error(func_divider(1, 1, na.rm = c(TRUE, FALSE)))
  
}
```
I named my unit test script `test_func_divider.R`. Now I can run my test by referencing that script:
```r {linenos=table,linenostart=1}
test_file("tests/test_func_divider.R")
```
```
== Testing test_func_divider.R ==============================
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 0 ] Done!
```
Fortunately, `func_divider()` passes all tests! Eventhough the code itself is reasonably robust as both the function
inputs and outputs are checked, we can still improve the robustness of the function by including function documentation. RStudio generates a [Roxygen2 skeleton](https://kbroman.org/pkg_primer/pages/docs.html) by pressing `CTRL` + `SHIFT` + `ALT` + `R` when the cursor is located inside a function.

```r {linenos=table,linenostart=1}
#' Title: A custom function that divides two numbers or two number vectors of 
#' Title: equal length
#' 
#' @author : MyName
#' @date: January 1, 1970
#'
#' @param numerator: A single number or vector of numbers, must be numeric
#' @param denominator: A single number or vector of numbers, must be numeric
#' @param na.rm : A boolean that specifies whether NaNs need to removed first.
#'                Takes only one argument, with a default of FALSE
#'
#' 
#' 
#'
#' @examples: func_divider(4, 2, na.rm = TRUE),
#'            func_divider(seq(0, 0, length.out = 100), seq(0, 0, length.out = 100))

func_divider <- function(numerator, denominator, na.rm = FALSE) {
  
  
  stopifnot(
            # Are numerator and denominator of equal length?
            length(numerator) == length(denominator),
            # Are numerator and denominator both numeric?
            is.numeric(numerator) & is.numeric(denominator),
            # Is the length of numerator and denominator greater than zero?
            length(numerator) > 0 & length(denominator) > 0,
            # Is input for na.rm a Boolean and length 1?
            is.logical(na.rm), length(na.rm) == 1
            )
  
  # Does the denominator vector contain a zero?
  #If so, print an warning message
  if(0 %in% denominator) {
    warning("Dividing by zero is undefined")
  }

  # Remove NA's from both input vectors when na.rm is set to TRUE.
  if (na.rm == TRUE) {
    miss <- is.na(numerator) | is.na(denominator)
    numerator <- numerator[!miss]
    denominator <- denominator[!miss]
  }
  
  fraction <- numerator/denominator
  return(fraction)
}
```
In conclusion, writing more robust functions in R is easy. Checking function inputs
is done by including checks within the function itself, while testing function output
is done outside of the function by writing unit tests. 
