---
title: Query Language Function Types
keywords: query language
tags: [query language]
sidebar: doc_sidebar
permalink: query_language_function_types.html
summary: Understand how to use types of functions (aggregation, time window, etc.).
---
When you get started with Wavefront Query Language, it helps to understand the different types of functions and what they do.

## Aggregation Functions

An [aggregation function](query_language_reference.html#aggregation-functions) returns a series of points whose values are calculated from corresponding points in two or more input time series. For example, you could request the sum, average, or maxiumum of the number of bytes received from multiple sources.

Here's what you need to know:

* An aggregation function calculates an aggregation only if there's at least 1 point present.
* The function can consolidate into a single value or into multiple values by using a group-by clause. For example `avg(ts(~sample.network.bytes.recv), az)` returns the average for all time series, but groups the result by availability zone (az).
* **Standard aggregation functions** are great for aggregating data that is not perfectly aligned. They apply interpolated values if necessary when 1 or more values are reported at a given point in time.
* **Raw aggregation functions** to not perform interpolation. They are great if data is already aligned, or if you really want the raw data.

See [Aggregating Time Series](query_language_aggregate_functions.html) for detailed examples and a lightboard video.

## Moving Time Window Functions

A [moving time window function](query_language_reference.html#moving-window-time-functions) performs vertical aggregation by combining the values of a single time series over a sliding time window. Each function includes a <timeWindow> argument, which is the amount of time in the moving time window. You can specify a time measurement based on the clock or calendar (1s, 1m, 1h, 1d, 1w), the window length (1vw) of the chart, or the bucket size (1bw) of the chart. Default is minutes if the unit is not specified.

Functions include `mavg()`, which returns the moving average for each series, or `mcount()` which returns the number of data points reported by each series.

Examples include finding the lowest value for a metric in the last 24 hours, and examining if a source reported a metric the number of times expected in the last 2 hours.

See [Using Moving and Tumbling Windows to Highlight Trends](query_language_windows_trends.html) for details and examples, and see [Queries with a Time Focus](query_language_recipes.html#queries-with-a-time-focus) for more examples of both standard time functions and moving time window functions.

## Filtering and Comparison Functions

 
