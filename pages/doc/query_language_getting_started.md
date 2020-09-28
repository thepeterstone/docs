---
title: Query Language Quickstart
keywords: query language
tags: [query language, getting started, videos]
sidebar: doc_sidebar
permalink: query_language_getting_started.html
summary: Watch some videos, run a query, apply filters and functions, and more.
---
The Wavefront Query Language lets you retrieve and display the data that has been ingested into Wavefront.
* **Time series data** The query language is particularly well suited to time series data, because it accommodates the periodicity, potential irregularity, and streaming nature of that data type.
* **Histograms** The query language includes functions for [manipulating histograms](query_language_reference.html#histogram-functions).
* **Traces and spans** Use the [tracing UI](tracing_ui_overview.html) to query traces and spans.

This page uses the v2 UI, which allows you to examine your data with [chart builder](chart_builder.html) and perform advanced exploration with [query editor](query_editor.html).

Watch these videos to get you started. The videos use the v1 UI, but the basic workflow remains the same in the v2 UI.

<table style="width: 100%;">
<tbody>
<tr><td width="50%"><a href="https://vmwarelearningzone.vmware.com/oltpublish/site/openlearn.do?dispatch=previewLesson&id=60b992dc-dc7a-11e7-a6ac-0cc47a352510&inner=true&player2=true"><img src="/images/v_ql_intro.png" alt="introduction to query language"/></a></td>
<td width="50%"><a href="https://vmwarelearningzone.vmware.com/oltpublish/site/openlearn.do?dispatch=previewLesson&id=61f9391c-dc7a-11e7-a6ac-0cc47a352510&inner=true&player2=true"><img src="/images/v_ql_basics.png"/></a></td></tr>
</tbody>
</table>

## Intro: Anatomy of a Query

Before you run your first query, let's look at the anatomy of a query shown in Chart Builder:

![annotated chart builder, items discussed below](images/query_anatomy_builder.png)

Each query has the following components. Only the metric is required, the other elements are optional but help you get the information you're really interested in.
* A metric (or a constant such as `10`). In this example, `~sample.cpu.loadavg.1m`
* One or more sources, that is, host, VM, container, etc. In this example, `app-*`. That means metrics that come from `db-*` are ignored.
* One or more point tags. In this example, `env=production`. Point tags must be defined in your time series. If you use Chart Builder, only valid point tags are available for selection.
* One or more functions. This example uses the `avg()` function, and the `mmedian()`` function with a 10 minute time window. The [Query Language Reference](query_language_reference.html) lists each function with a short description and points to reference pages.

Here's the same query in the Query Editor.

![annotated query editor, items discussed above](images/query_anatomy_editor.png)


## Step 1: Retrieve a Metric

The Chart Builder UI makes it easy to show any metric that's currently flowing into your Wavefront instance. Let's explore some sample data, which are included with each Wavefront instance.

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">
<ol>
<li>Log in to your Wavefront instance, which has a URL &lt;my_instance&gt;.wavefront.com </li>
<li>Select <strong>Dashboards > New Chart</strong>.</li>
<li>In the Chart Builder, select the metric ~sample.cpu.loadavg.1m. Autocomplete helps with the selection. </li></ol>
</td>
<td width="50%">
<img src="images/chart_builder_autocomplete.png" alt="Zoom in on data selection in chart builder, showing auto-complete."></td>
</tr>
</tbody>
</table>

Next, explore adding ~sample metrics. If you like, switch to Query Editor and add a constant -- but note that you can't switch back to Chart Builder!

Here's an annotated screenshot of the first chart you'll see.

* **Chart names** are easy to change just by typing.
* For quick zoom in/out, use the **hover time selector**, which appears when the cursor is on the chart.
* As you zoom in or out, the [bucket size (chart resolution)](ui_charts.html#chart-resolution) changes.
* Use **Share chart** or **Quick share** to [share with others](ui_sharing.html).
* Use the Query Editor toggle for some advanced query functionality
* Notice [events](events.html) that are shown on the time line. These events are often system events associated with alerts, but can be user-defined events.
* Be sure to **Save** the chart, either to an existing or a new dashboard.

![First simple query shown in annotated chart. Items are explained in text above. ](images/query_quickstart_first_query.png)

**Things to Try**

* Use the Hover Time Selector to zoom in and out. You can also select-drag to see part of the chart, then click + or - to return to default settings.
* Hover over event icons in the Y axis to get details for the event.
* Hover over a time series to see the legend. Use Shift P to pin the legend.

## Step 2: Filter by Source and Point Tag

The example chart is quite busy, but we can use filters to focus in.

<table style="width: 100%;">
<tbody>
<tr>
<td width="60%">
1. Make sure <strong>Data</strong> is still ~sample.cpu.loadavg.1m. </td>
<td width="40%"> </td></tr>
<tr>
<td>2. Click <strong>Filters</strong>, select <strong>source</strong>, and type <strong>app-&#42;</strong> to include only time series if the source name starts with <strong>app-</strong>. This query uses a wildcard character.</td>
<td><img src="images/query_quickstart_source.png" alt="Add source to Filter"></td>
</tr>
<tr>
<td>
3. Click the <strong>Add</strong> botton and select <strong>env &gt; production</strong> as the second filter.
</td>
<td width="50%">
<img src="images/query_quickstart_env.png" alt="Select env=production">
</td>
</tr>
</tbody>
</table>

**Things to Try**

* Explore the effect of using different source and point tag filters.
* Add more than one filter for each category, for example, several sources.
* Clone a query and click the Query Editor toggle `</>` to see the results in Query Editor (you can't return to Query Builder, so using a clone helps.)
* With multiple queries in place, show and hide queries, and drag them to change query order.

## Step 3: Apply an Aggregation Function

[Aggregation functions](query_language_aggregate_functions.html) allow you to combine points from multiple time series, and to group the results. Let's take the average first, and then let's remove the `env` filter and instead group by environment.

<table style="width: 100%;">
<tbody>
<tr>
<td width="60%">
1. Make sure <strong>Data</strong> is still ~sample.cpu.loadavg.1m. </td>
<td width="40%"> </td></tr>
<tr>
<td>2. Click <strong>Functions</strong>, and pick <strong>Favorites &gt; avg</strong>. The result is a single aggregated time series.

In Query Editor, this query looks like this:
<p><code>sum(ts(~sample.cpu.loadavg.1m))</code></p>
</td>
<td><img src="images/query_quickstart_avg.png">
</td>
</tr>
<tr>
<td>
3. Click <strong>Functions &gt; Favorites &gt; avg</strong> again and select <strong>Group by</strong> and then <strong>env</strong>.

The result is two aggregated time series. You can hover over each line to see which environment it shows.

In the Query Editor, you can add the literal <strong>, pointTags</strong> (you need the comma!), so the query looks like this:
<p><code>sum(ts(~sample.cpu.loadavg.1m), pointTags)</code></p>
</td>
<td>
<img src="images/query_quickstart_group_by.png" alt="Select env=production">
</td>
</tr>
<tr>
<td>
Add a second function. For example you can use the deriv() function to show the rate of change per second for the sum.
<p><code>deriv(sum(ts(~sample.cpu.loadavg.1m))</code></p> </td>
<td><img src="/images/v2_quickstart_deriv.png" alt="apply second function in chart builder"></td>
</tr>
</tbody>
</table>

**Things to Try**

Experiment with some of our other functions, either in Chart Builder or in Query Editor.

* Use one of the [Moving Window Time Functions](query_language_reference.html#moving-window-time-functions) to combine or test the values of a time series over a time sliding window.
* Experiment with [Filtering and Comparison Functions](query_language_reference.html#filtering-and-comparison-functions). For example, use `topk()` to return the top `numberOfTimeSeries` series.


<!---
## Step 4: Further Chart Customization

The [query language](query_language_reference.html) supports many other ways of getting just the results you want from your data. Here are some examples;

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
Add a second function. For example we can use the deriv() function to show the rate of change per second for the sum.
<p><code>deriv(sum(ts(~sample.requests.total.num))</code></p> </td>
<td width="60%"><img src="/images/v2_quickstart_deriv.png" alt="create dashboard"></td>
</tr>
<tr>
<td width="40%">
You can also add a second filter. Our sample data include an env and an az point tag, and we can select one of the values of that tag. </td>
<td width="60%"><img src="/images/v2_quickstart_tag.png" alt="group by tag"></td>
</tr>
</tbody>
</table>
--->



## Next Steps

What's next depends on the type of data you're interested in, and how you want to interact with your data.

### Query Types for Different Data

Most Wavefront users query for time series metrics, but we support interacting with other data.

Charts for metrics also support the following types of queries:
* **Events**: Query Wavefront events with [`events()` queries](query_language_reference.html#event-functions).
* **Histograms**: Query histograms with [`hs() queries`](visualize_histograms.html#querying-histogram-metrics)
* **Traces and spans**: Query trace data from the tracing UI with the [tracing Query Builder](trace_data_query.html)

### Docs, Videos, and Query Language Recipes

Wavefront documentation includes videos, tutorials, reference, and guides on the query language. 

- **[Query Language Videos](videos_query_language.html)** get you started and [Use Case Videos](wavefront_use_cases.html) show off some compelling examples.
- **[Query builder](query_language_query_builder.html)** (for v1) and **[Chart builder](chart_builder.html)** (for v2) can help you come up to speed quickly while using the product.
- If you're logged in to Wavefront, select **Integrations** in the task bar and find the **Tutorial** or the **Tour Pro** integration. The Tutorial includes an Interactive Query Language Explorer that shows examples for each function.
- [Wavefront Query Language Reference](query_language_reference.html) lists each function and gives query language syntax element. Each function names is a link to a reference page for the function.
- For in-depth discussions and examples, we have a **[reference page](label_query%20language.html)** for each function and some [Query Language Recipes](query_language_recipes.html).

## FAQ

This doc set includes videos and explanations from the engineering team that helps you come up to speed quickly:

<table style="width: 100%;">
<tbody>
<thead>
<tr><th width="40%">Question</th><th width="30%">Doc/Blog</th><th width="30%">Video</th></tr>
</thead>
<tr>
<td>How can I combine multiple series?</td>
<td markdown="span">[Aggregating Time Series](query_language_aggregate_functions.html) </td>
<td markdown="span">[Time Series and Interpolation](https://youtu.be/9LnDszVrJs4) </td></tr>
<tr>
<td>Why does my query return NO DATA?</td>
<td markdown="span">Maybe the time series don't match. See [When Multiple Series Match (Or Not)](query_language_series_matching.html) </td>
<td> </td></tr>
<tr>
<td>I got a warning about pre-aligned data. Why? </td>
<td markdown="span">Wavefront improves performance by wrapping `align()` around certain functions. See [Bucketing with align()](query_language_align_function.html) </td>
<td> </td></tr>
<tr>
<td>How can I use Wavefront for anomaly detection?</td>
<td markdown="span">You can use [AI Genie](ai_genie.html) or [detect anomalies with functions and statistical functions](query_language_statistical_functions_anomalies.html). </td>
<td><ul><li><a href="https://youtu.be/XiSkNETTfCI">AI Genie Anomaly Detection</a></li>
<li><a href="https://youtu.be/I-Z9d94Zi7Y">Anomaly Detection with Functions</a></li></ul> </td></tr>
<tr>
<td>How can I improve query performance?</td>
<td markdown="span">Consider [bucketing with align()](query_language_align_function.html).
Investigate [slow queries](wavefront_monitoring.html#examine-slow-queries).</td> <td> </td></tr>

</tbody>
</table>

<!---
<tr>
<td>How do time windows work?</td>
<td markdown=span>Wavefront supports [moving time window functions](). </a>.
Investigate <a href="https://docs.wavefront.com/dashboards_slow_queries.html">slow queries</a>.</td><td> </td></tr>
<tr>
<td>How do I calculate the moving averate over a set of time (e.g. 24 hours)?</td>
<td markdown=span>Use a moving time window function. See [Calculating Continuous Aggregation with Moving Window Functions](query_language_windows_trends.html#calculating-continuous-aggregation-with-moving-window-functions).</td><td> </td></tr>
<tr>
<td>How do I calculate over a specified of time (e.g. daily average)?</td>
<td markdown=span>Use a tumbling time window. See [Tumbling Windows Examples](query_language_windows_trends.html#tumbling-window-examples).</td><td> </td></tr>
--->
