---
title: Standard Deviation Function
keywords: query language reference
tags: [reference page]
sidebar: doc_sidebar
permalink: hs_stddev.html
summary: Reference to the stddev() function
---
## Summary
```
stddev(<hsExpression>)
```

Standard deviation tells you how the data in the histogram expression is distributed around the mean, and returns a composite histogram distributions.


## Parameters


<table style="width: 100%;">
<thead>
<tr><th width="30%">Parameter</th><th width="70%">Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">[hsExpression](query_language_reference.html#query-expressions)</td>
<td markdown="span">Expression describing the histogram series to be merged.</td></tr>
</tbody>
</table>


## Description

The `stddev()` functions shows you how the data in a histogram expression varies agains the mean or average and returns those values as time series. A separate time series is returned for each histogram series. You can use find out the anomalies in your histogram expression using this function.

## Example

This chart represents all of the histogram series described by the expression `hs(tracing.derived.*.duration.micros.m))`. Each histogram series consists of distributions from a particular source, and a given source might emit more than one histogram series. The chart represents each histogram series as a separate line that consists of the median values of the distributions.

![hs_stddedv_before](images/hs_stddedv_before.png)

**Example : Finding the standard deviation**

Here we include all of the histogram series in the results:
 
```
stddev(hs(tracing.derived.*.duration.micros.m)))
```

Now you see how the data in your expression varies against the mean. You see an anomaly (a spike) between 9.55 AM and 10 AM.

![hs_stddev](images/hs_stddev.png)


## See Also

* [Wavefront Histograms](proxies_histograms.html)
