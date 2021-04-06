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
lastmod: "2021-04-06T00:00:00Z"

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

## Overview

```r
# A gist for making custom functions more robust
library(testthat)

# testthat provides a set of functions to check whether the result of the function
# is as expected

# A simple function with no warning or error messaging
func_divider <- function(numerator, denominator, na.rm = FALSE) {
  
  stopifnot(is.logical(na.rm), length(na.rm) == 1)  # is input for na.rm a Boolean and length 1?
  stopifnot(length(numerator) == length(denominator))         # Are numerator and denominator of equal length?
  stopifnot(is.numeric(numerator) & is.numeric(denominator))  # Are numerator and denominator both numeric?
  
  if (na.rm == TRUE) {
    miss <- is.na(numerator) | is.na(denominator)
    numerator <- numerator[!miss]
    denominator <- denominator[!miss]
  }
  
  numerator / denominator
}

func_divider(2,4)
func_divider(0,0) # By default division by zero returns NaN

# Write a unit test for func_divider
test_divider <- function() {
  
  expect_equal(func_divider(4, 2), 2) # 4/2 == 2
  expect_equal(func_divider(4, 0), Inf) # 4/0 == Inf
  expect_equal(func_divider(0, 0), NaN) # 4/0 == Inf
  expect_type(func_divider(4, 2), "double")
  expect_gt(func_divider(4, 2), func_divider(2, 2)) # 4/2 > 2/2
  expect_lte(sum(func_divider(1:4, 4:1)),  # Can handle a vector as input, but can expects a
             sum(func_divider(4:1, 1:4)))  # single logical value for the comparison
  
}

test_divider()


# Now that the code itself is reasonably robust by checking both the function
# inputs and testing its output, we can still improve the function by including
# function documentation. RStudio provides a template by pressing
# ctrl + shift + alt + r when the cursor is located inside a function

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
  
  stopifnot(is.logical(na.rm), length(na.rm) == 1)  # is input for na.rm a Boolean and length 1?
  stopifnot(length(numerator) == length(denominator))         # Are numerator and denominator of equal length?
  stopifnot(is.numeric(numerator) & is.numeric(denominator))  # Are numerator and denominator both numeric?
  
  if (na.rm == TRUE) {
    miss <- is.na(numerator) | is.na(denominator)
    numerator <- numerator[!miss]
    denominator <- denominator[!miss]
  }
  
  numerator / denominator
}

```
