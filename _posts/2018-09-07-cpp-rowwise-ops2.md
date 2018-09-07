---
layout: default_post
title: (Follow-up) Using C++ to Speedup Row-wise Operations
date: 2018-09-07
description: More on the R package Rcpp, code performance, and for loops.  
image: /images/rcpp_blog2.svg
tags: [C++, R]
---

### Follow-Up
A curious colleague pointed out the strongly asserted internet claim that R For Loops and R Apply functions ought to be close peers when it comes to performance. Some example sources: <a href="https://stackoverflow.com/questions/2275896/is-rs-apply-family-more-than-syntactic-sugar">here</a> and <a href="https://support.rstudio.com/hc/en-us/articles/218221837-Profiling-with-RStudio">here</a>. However, my previous <a href="https://chrisdienes.github.io/blog/2018/08/22/cpp-rowwise-ops">post</a>.  constructed an experiment which suggested a huge performance gain using an `apply()` function over `for(i in …)`. I noticed my R For Loop was constructed to mimic the C++ function I had written, and not the R `apply()` function. I re-ran the test as before, but this time including two additional alternatives. The new code and results are given below.  

### New Code     

Refer to the initial <a href="https://chrisdienes.github.io/blog/2018/08/22/cpp-rowwise-ops">post</a> for the missing details. Below is a new R For Loop which more closely mimics the behavior of our `apply()` procedure.  

```r
na_or_0_row_ind = function(r){all(is.na(r)) | all(r == 0)}
stm <- Sys.time()
  forloop_rows_v2 <- vector(mode = "logical", length = nrow(df))
  for(rr in 1:nrow(df)){forloop_rows_v2[rr] <- na_or_0_row_ind(df[rr,])}
  forloop_rows_v2 <- which(forloop_rows_v2)
forloop_time_v2 <- as.numeric(difftime(Sys.time(),stm, units = "secs"))
```

Below we use `vapply()` which requires a transpose operation in our underlying use case. If we could avoid the transpose, we observed slightly improved results over `apply()`. Yet with the transpose we had slower performance.

```r
stm <- Sys.time()
  df2 <- data.frame(t(df))
  vapply_rows <- which(vapply(df2, na_or_0_row_ind, logical(1)))
vapply_time <- as.numeric(difftime(Sys.time(),stm, units = "secs"))
```

 
### New Results

The below times and time ratios were obtained using Microsoft R Open 3.4.3 (which is the R version tied to the use case). Once again, I only did one run per treatment <a href="https://www.youtube.com/watch?v=bFEoMO0pc7k">because…</a>.  

| Rows                | Cpp<br>Time     | Apply<br>Time      | Vapply Time  | R Loop (Apply) Time | R Loop (C++) Time | 
| :-----------------: | :------------: | :---------------: | :----------: | :-----------------: | :---------------: |
| 6,000               | 0.004          | 0.068             | 0.097        | 2.067               | 3.519             | 
| 60,000	            | 0.011          | 0.502             | 0.761        | 27.264              | 36.536            |
| 600,000	            | 0.088          | 3.317             | 5.953        | 913.684             | 1516.065          | 
| 6,000,000           | 0.897          | 38.057            | 60.063       | DNR                 | DNR               |
| 60,000,000          | 9.946          | 434.300           | 836.376      | DNR                 | DNR               |

| Rows             | Cpp Ratio     |  Apply Ratio     | Vapply Ratio | R Loop (Apply) Ratio | R Loop (C++) Ratio | 
| :-----------------: | :------------: | :---------------: | :----------: | :------------------: | :----------------: |
| 6,000               | 1.0            | 17.0              | 24.2         | 516.0                | 878.2              | 
| 60,000	            | 1.0            | 45.6              | 69.2         | 2476.5               | 3318.8             |
| 600,000	            | 1.0            | 37.6              | 67.6         | 10371.4              | 17209.2            | 
| 6,000,000           | 1.0            | 42.4              | 67.0         | DNR                  | DNR                |
| 60,000,000          | 1.0            | 43.7              | 84.1         | DNR                  | DNR                |

### Summary

The R For Loop did see a performance improvement when coded more in-line with the `apply()` operation, yet it was still noticeably slower. Interestingly, our C++ and R Apply operations tend to scale almost linearly, but the For Loops became unstable after 60K rows. I'm curious if this generalizes to other people's hardware?  
