---
layout: default_post
title: The low-rank hurdle model
date: 2017-09-08
description: A new low-rank modeling framework for data with special values.   
image: /images/low_rank_blog.svg
tags: [Paper, Research]
---

I recently wrote a paper  entitled <a href="" target="_blank">The low-rank hurdle model</a>. Now you may be wondering what's a low-rank hurdle model? If you don't want to read the entire paper, I'll try to provide a simple explanantion below.

Often data scientists have the luxury of working with large and diverse data sets. The volume and variety of data ensures their jobs will continue to be new, exciting, and (hopefully) durable against automation for years to come. But sometimes this luxury is a curse. Sometimes the excitement for new, large, and diverse data fades as we struggle to match existing methodologies with the challenges at hand. Of course this is a call for innovation.

One of the classic approaches to handling large data sets is to reduce its size. In particular, a researcher may try to perform <a href="https://en.wikipedia.org/wiki/Feature_selection" target="_blank">variable selection</a> or <a href="https://en.wikipedia.org/wiki/Feature_extraction" target="_blank">feature extraction</a> to reduce the overall number of variables. One of the more popular approaches is to use principal component analysis (<a href="https://en.wikipedia.org/wiki/Principal_component_analysis" target="_blank">PCA</a>) which constructs a low-dimensional approximation of the original data set. This is sometimes referred to as a low-rank approximation. PCA’s popularity is aided by its mathematical conveniences and interpretability. Unfortunately, PCA requires data to be numerical; and additionally, PCA measures the quality of approximation using a metric known as mean squared error (<a href="https://en.wikipedia.org/wiki/Mean_squared_error" target="_blank">MSE</a>) which is not appropriate for all data types. Specifically, PCA will struggle representing variables which are categorical, skewed, counts, zero-inflated, imbalanced...

So what does this mean for data scientists? Remember all that data variety which makes the job so great? Well, it’s not friends with PCA.

Other researchers have identified and resolved some of the problems with PCA. The typical approach is to construct a better metric for measuring the quality of the data approximation. So instead of optimizing with respect to MSE, you choose a <a href="https://en.wikipedia.org/wiki/Loss_function" target="_blank">loss function</a> which better describes the underlying data structure. The existing modifications are typically described under the "generalized low-rank model" and a great summary can be found <a href="https://web.stanford.edu/~boyd/papers/pdf/glrm.pdf" target="_blank">here</a>.

Unfortunately the existing literature is incomplete and lacked specialized approaches for <a href="https://en.wikipedia.org/wiki/Zero-inflated_model" target="_blank">zero-inflated</a> variables. Zero-inflated count variables are commonly found in defect count data sets which are typical in the manufacturing industry, for example. To improve the methodology for such variables, I proposed using the hurdle loss function which is all explained in the <a href="" target="_blank">paper</a>.  Now before you go diving-in head first, I’ll warn you the paper requires some mathematical and statistical background and is not light bedtime reading. So as the old proverb reminds us: forewarned is forearmed...

The proposed methodology may be of interest to researchers in several fields of study. The approach is an expansion of the <a href="https://genomebiology.biomedcentral.com/articles/10.1186/s13059-015-0805-z" target="_blank">ZIFA model</a> which has applications for genomics. Additionally, the hurdle model outperforms popular <a href="https://en.wikipedia.org/wiki/Missing_data" target="_blank">missing value</a> methods for a particular class of imputation problem. This last contribution may provide much needed insights to fields such as survey analysis where missing values can be common.
