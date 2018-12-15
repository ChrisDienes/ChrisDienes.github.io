---
layout: default_post
title: Using C++ to Speedup Row-wise Operations
date: 2018-08-22
description: The R package Rcpp is a great tool for improving your code’s performance.  
image: /images/rcpp_blog.svg
tags: [C++, R]
---

### Summary
Data analysis code often falls into the ad hoc class: a one-time use which is destined for the graveyard of forgotten files that are slowly decomposing your computer’s available storage space. But some code falls into the reusable class; a category which requires thoughtful considerations of robustness, maintenance, and performance. Recently I encountered a performance bottleneck while productionalizing a particular workflow. The original code was written in R and contained several time consuming operations. Specifically, data preprocessing involved ingesting a large but narrow data set (rows: 60+ million, columns: around 15), and then flagging any row which contained either all zeros or all NAs. Loop-ing and apply-ing over rows can be a very time consuming task within R. Fortunately, 1) C++ is very efficient at these operations, and 2) the Rcpp package provides a simple way to integrate your C++ code within your R workflow. For a more detailed Rcpp tutorial see <a href="http://adv-r.had.co.nz/Rcpp.html">here</a>.
 
### Results
Below I compare the performance of three different row-wise paradigms: an R `for` loop, an R `apply`, and a C++ `for` loop integrated into R using Rcpp. The row-wise task is to find all row index values which fail the zero/NA check (described above), where our narrow data set has 15 columns and N rows. Here N is varied by factors of 10 from 6K up to 60 million. The number of rows with all zeros or all NAs are both set to 25% of the number of rows N. Note: Other approaches involving column-wise looping, recoding, and summarizing using row sums could also be explored, but the intent of this post is to provide a template for performing row-wise operations and comparing performance of row-wise alternatives.   
The below table contains the run times (in seconds) from one experiment per setting. Of course we ought to run the experiment multiple times to account for system variability but “ain’t nobody got time for that”. The “DNR” label stands for did not run. 

<div style = "text-align:center;overflow-x:auto;">
     <table style="margin: 0 auto;border-collapse:collapse;width: 100%;text-align:center;">
      <tr>
        <th>Rows</th>
        <th>Cpp Time</th>
        <th>Apply Time</th>
        <th>R Loop Time</th>
      </tr>
      <tr>
        <td>6,000</td>
        <td>0.003</td>
        <td>0.054</td>
        <td>4.285</td>
      </tr>
      <tr>
        <td>60,000</td>
        <td>0.010</td>
        <td>0.445</td>
        <td>41.096</td>
      </tr>
      <tr>
        <td>600,000</td>
        <td>0.094</td>
        <td>3.498</td>
        <td>1752.259</td>
      </tr>
      <tr>
        <td>6,000,000</td>
        <td>1.136</td>
        <td>35.517</td>
        <td>DNR</td>
      </tr>
      <tr>
        <td>60,000,000</td>
        <td>10.321</td>
        <td>397.695</td>
        <td>DNR</td>
      </tr>                                                                                         
     </table>
</div>

The below table contains the ratio of run time relative to the Cpp run time. Interestingly, Cpp is many times more efficient than the other two methods but is logically constructed in the same way as the R for loop. 

<div style = "text-align:center;overflow-x:auto;">
     <table style="margin: 0 auto;border-collapse:collapse;width: 100%;text-align:center;">
      <tr>
        <th>Rows</th>
        <th>Cpp Ratio</th>
        <th>Apply Ratio</th>
        <th>R Loop Ratio</th>
      </tr>
      <tr>
        <td>6,000</td>
        <td>1.0</td>
        <td>18.1</td>
        <td>1438.7</td>
      </tr>
      <tr>
        <td>60,000</td>
        <td>1.0</td>
        <td>44.5</td>
        <td>4106.8</td>
      </tr>
      <tr>
        <td>600,000</td>
        <td>1.0</td>
        <td>37.2</td>
        <td>18629.9</td>
      </tr>
      <tr>
        <td>6,000,000</td>
        <td>1.0</td>
        <td>31.3</td>
        <td>DNR</td>
      </tr>
      <tr>
        <td>60,000,000</td>
        <td>1.0</td>
        <td>38.5</td>
        <td>DNR</td>
      </tr>                                                                                         
     </table>
</div>

### Code     

**Step 1.**  Setup R session and compile C++ function for use within R:

 ```r
### You may need more digits for test times for small test sizes:
options(digits.secs = 6)

### ---------------------------------------------- ###
### load packages:
### ---------------------------------------------- ###

library(Rcpp)

### ---------------------------------------------- ###
### Compile C++ function
### ---------------------------------------------- ###

# Here we compile cpp code inside current session, but 
# could be built into a package for production purposes:

### Using in script compiling of c++ function:
cppFunction('std::vector<int> allNa_or_allZero(Rcpp::NumericMatrix x) {
            // find row count of numeric matrix
            int nRows = x.nrow();
            // using std for integer vector so I can call .reserve()
            std::vector<int> myvector;
            // preallocating the maximum memory for the object
            myvector.reserve(nRows);
            // loop over all rows of numeric matrix
            for (int i = 0; i < nRows; i++) {
              if (Rcpp::is_true(Rcpp::all(Rcpp::is_na( x(i, _) ) ) )) {
                // if all were NA then add to back of current vector
                myvector.push_back(i + 1);
              } else if ( Rcpp::is_true(Rcpp::all( x(i, _)  == 0) ) ) {
                // if all were 0 then add to back of current vector
                myvector.push_back(i + 1);
              }
            }
            return myvector;
            }')

### If compiling c++ function from a file (e.g. allNa_or_allZero.cpp) use:
# sourceCpp(".../allNa_or_allZero.cpp")

### Example file content:
#
# #include <Rcpp.h>
# using namespace Rcpp;
# 
# // [[Rcpp::export]]
# std::vector<int> allNa_or_allZero(Rcpp::NumericMatrix x) {
#   // find row count of numeric matrix
#   int nRows = x.nrow();
#   // using std for integer vector so I can call .reserve()
#   std::vector<int> myvector;
#   // preallocating the maximum memory for the object
#   myvector.reserve(nRows);
#   // loop over all rows of numeric matrix
#   for (int i = 0; i < nRows; i++) {
#     if (Rcpp::is_true(Rcpp::all(Rcpp::is_na( x(i, _) ) ) )) {
#       // if all were NA then add to back of current vector
#       myvector.push_back(i + 1);
#     } else if ( Rcpp::is_true(Rcpp::all( x(i, _)  == 0) ) ) {
#       // if all were 0 then add to back of current vector
#       myvector.push_back(i + 1);
#     }
#   }
#   return myvector;
# }
```

**Step 2.** Create some fake data:

```r
### ---------------------------------------------- ###
### create some fake data:
### ---------------------------------------------- ###

# Data set size:
N <- 6*10^3 # 6*10^3 6*10^4, 6*10^5, 6*10^6, 6*10^7  
df <- as.data.frame(matrix(1, ncol = 15, nrow = N))
# Create 25% all 0:
df[1:(N/4),] <- 0
#  Create 25% all NA:
df[(N/4 + 1):(N/2),] <- NA
``` 

**Step 3.** Run the three competing approaches:

```r
### ---------------------------------------------- ###
### use apply:
### ---------------------------------------------- ###

# custom function:
na_or_0_row_ind = function(r){all(is.na(r)) | all(r == 0)}
stm <- Sys.time()
apply_rows <- which(apply(df, 1, na_or_0_row_ind))
apply_time <- as.numeric(difftime(Sys.time(),stm, units = "secs"))

### ---------------------------------------------- ###
### use for loop:
### ---------------------------------------------- ###

if(N < 600001){
  stm <- Sys.time()
  counter <- 0
  forloop_rows <- rep(NA, N)
  for(rr in 1:N){
    test <- (all(is.na(df[rr,])) | all(df[rr,] == 0))
    if(test){
      counter <- counter + 1
      forloop_rows[counter] <- rr
    }
  }
  forloop_rows <- forloop_rows[1:counter]
  forloop_time <- as.numeric(difftime(Sys.time(),stm, units = "secs"))
}

### ---------------------------------------------- ###
### use Rcpp:
### ---------------------------------------------- ###

stm <- Sys.time()
cpp_time <- cpp_rows <- allNa_or_allZero(as.matrix(df))
cpp_time <- as.numeric(difftime(Sys.time(),stm, units = "secs"))
``` 

**Step 4.** Compare results:

```r
### ---------------------------------------------- ###
### Compare Results
### ---------------------------------------------- ###

### compare rows found:
all.equal(forloop_rows, cpp_rows)
all.equal(apply_rows, cpp_rows)

### compare elapsed times:
apply_time
forloop_time
cpp_time

### compare elapsed time ratios
apply_time / cpp_time
forloop_time / cpp_time
```
