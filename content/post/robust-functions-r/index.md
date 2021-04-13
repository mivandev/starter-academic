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
# Creating more robust functions in R

In the code below, I demonstrate how easy it can be to improve the robustness
of custom functions in R. First, I'll show how the function inputs can be 
checked for validity and how including error and warnings into the function 
can lead to easier debugging of faulty code. Second, a demonstration of two ways
to check whether the function produces expected outcomes.

We will use a very simple function without error handling and without checking
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
```r {linenos=table,linenostart=1, echo = TRUE, error = TRUE}
func_divider(1:3, 1:2)
```
```r
Error in func_divider(1:3, 1:2) : 
  length(numerator) == length(denominator) is not TRUE
```

```r {linenos=table,linenostart=1}
```
```r {linenos=table,hl_lines=[8,"15-17"],linenostart=1}
```
