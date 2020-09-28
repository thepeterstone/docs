---
title: Distributed Tracing Key Concepts
keywords: data, distributed tracing
tags: [tracing]
sidebar: doc_sidebar
permalink: trace_data_details.html
summary: Get to know the concepts around distributed tracing in Wavefront.
---
Wavefront follows the [OpenTracing](https://opentracing.io/) and [OpenTelemetry](https://opentelemetry.io/) standards for representing and manipulating trace data.

## Traces 
A trace shows you how a request propagates from one microservice to the next in a distributed application. The basic building blocks of a trace are its spans, where each span corresponds to a distinct invocation of an operation that executes as part of the request.


This diagram illustrates a trace for a particular request that started with the Shopping service's `orderShirts` request and finished with the Delivery service's `dispatch` request.

![tracing trace spans](images/tracing_trace_spans.png)

* This trace consists of 9 member spans, one for each operation performed in the request. The span for the first request (namely, the Shopping service's `orderShirts` span) is the root span of the trace.

* Several of the spans in our sample trace have parent-child relationships to other spans in the trace. For example, the Styling service’s makeShirts span has two child spans (printShirts and giftWrap), and each of these spans has a child span of its own.

* A parent-child relationship exists between two spans when one operation passes data or control to another, either in the same service or in a different one.
A parent span with multiple children represents a request that invokes multiple operations, either serially or in parallel.
You can think of the trace as a tree of related spans. The trace has a unique trace ID, which is shared by each member span in the tree.

* Trace IDs are not normally displayed because they are long and hard to remember. For convenience, we refer to a trace by using the service and operation of its root span. This means we use shopping: orderShirts as the label for the entire trace, as well as for its root span.

* Different traces have the same label if they represent different calls to the same operation. For example, a new, separate trace begins every time the Shopping service’s orderShirts API is called. The trace in our example is just one of potentially thousands of traces that start with a call to orderShirts. Each such trace has a unique trace ID, and normally has a different start time and duration.

Spans are the fundamental units of trace data. This page provides details about the Wavefront format of a span, as well as the RED metrics that Wavefront automatically derives from spans. These details are mainly useful for developers who need to perform advanced customization.

## Spans

A Wavefront trace consists of one or more spans, which are the individual segments of work in the trace. Each span represents time spent by an operation in a service (often a microservice).

Spans are the fundamental units of trace data. This page provides details about the Wavefront format of a span, as well as the RED metrics that Wavefront automatically derives from spans. These details are mainly useful for developers who need to perform advanced customization.

### Wavefront Span Format

A well-formed Wavefront span consists of fields and span tags that capture span attributes. These attributes enable Wavefront to identify and describe the span, organize it into a trace, and display the trace according to the service and application that emitted it. Some attributes are required by the OpenTracing specification and others are required by Wavefront.

Most use cases do not require you to know exactly how Wavefront expects a span to be formatted:
* When you instrument your application with a [Wavefront OpenTracing SDK](wavefront_sdks.html#sdks-for-collecting-trace-data) or a [framework SDK](wavefront_sdks.html#sdks-that-instrument-frameworks), your application emits spans that are automatically constructed by the Wavefront Tracer. (You supply some of the attributes when you instantiate the [ApplicationTags](tracing_instrumenting_frameworks.html#application-tags) object required by the SDK.)
* When you instrument your application with a [Wavefront sender SDK](wavefront_sdks.html#sdks-for-sending-raw-data-to-wavefront), your application emits spans that are automatically constructed from raw data you pass as parameters.
* When you instrument your application with a 3rd party distributed tracing system, your application emits spans that are automatically transformed by the [integration](tracing_integrations.html#tracing-system-integrations) you set up.

It is possible to manually construct a well-formed span and send it either [directly to the Wavefront service](direct_ingestion.html#trace-data-spans) or to a TCP port that the Wavefront proxy is listening on for trace data. You might want to do this if you instrumented your application with a proprietary distributed tracing system.

### Span Syntax

```
<operationName> source=<source> <spanTags> <start_milliseconds> <duration_milliseconds>
```
Fields must be space separated and each line must be terminated with the newline character (\n or ASCII hex 0A).

For example:
```
getAllUsers source=localhost traceId=7b3bf470-9456-11e8-9eb6-529269fb1459 spanId=0313bafe-9457-11e8-9eb6-529269fb1459 parent=2f64e538-9457-11e8-9eb6-529269fb1459 application=Wavefront service=auth cluster=us-west-2 shard=secondary http.method=GET 1552949776000 343
```

### Span Fields

<table id = "spanfields">
<colgroup>
<col width="25%" />
<col width="10%" />
<col width="35%" />
<col width="30%" />
</colgroup>
<thead>
<tr>
<th>Field</th>
<th>Required</th>
<th>Description</th>
<th>Format</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">`operationName`</td>
<td>Yes</td>
<td>Name of the operation represented by the span.</td>
<td>String of less than 1024 characters. <br>Valid: a-z, A-Z, 0-9, hyphen ("-"), underscore ("_"), dot (".").</td>
</tr>
<tr>
<td markdown="span">`source`</td>
<td>Yes</td>
<td>Name of a host or container on which the operation executed.</td>
<td>String of less than 1024 characters.<br>Valid: a-z, A-Z, 0-9, hyphen ("-"), underscore ("_"), dot ("."). </td>
</tr>
<tr>
<td markdown="span">`spanTags`</td>
<td>Yes</td>
<td markdown="span">See [Span Tags](#span-tags), below. </td>
<td></td>
</tr>
<tr>
<td markdown="span">`start_milliseconds`</td>
<td>Yes</td>
<td>Start time of the span, expressed as Epoch time. </td>
<td markdown="span">Whole number of Epoch milliseconds [or other units (see below)](#time-value-precision-in-spans). </td>
</tr>
<tr>
<td markdown="span">`duration_milliseconds`</td>
<td>Yes</td>
<td>Duration of the span.</td>
<td markdown="span">Whole number of milliseconds [or other units (see below)](#time-value-precision-in-spans). Must be greater than or equal to 0. </td>
</tr>
</tbody>
</table>

### Span Tags

Span tags are special tags associated with a span and are key-value pairs.

- **Required**. Many of the span tags are required for a span to be valid.
- **Optional (Custom)**. An application can be instrumented to include custom span tags as well. Custom tag names must not use the reserved span tag names.

{% include note.html content="Make sure your span tags are within the maximum character limit as explained below." %}
The following table lists the maximum number of characters you can assign a span tag.
<table>
<colgroup>
<col width="20%" />
<col width="10%" />
<col width="70%" />
</colgroup>
<thead>
<tr>
<th>Element</th>
<th>Maximum Length</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>Span tag key</td>
<td>128</td>
<td>If the span tag key exceeds the maximum length, the span associated with it is blocked by Wavefront.</td>
</tr>
<tr>
<td>Span tag value</td>
<td>128</td>
<td>If the span tag value exceeds the maximum length, the value is truncated to the maximum number of characters.</td>
</tr>
</tbody>
</table>

The following table lists span tags that contain information about the span's identity and relationships.

<table>
<colgroup>
<col width="20%" />
<col width="10%" />
<col width="55%" />
<col width="15%" />
</colgroup>
<thead>
<tr><th>Span Tags <br>for Identity</th><th>Required</th><th>Description</th><th>Type</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`traceId`</td>
<td markdown="span">Yes</td>
<td markdown="span">Unique identifier of the trace the span belongs to. All spans that belong to the same trace share a common trace ID. </td>
<td markdown="span">UUID</td>
</tr>
<tr>
<td markdown="span">`spanId`</td>
<td markdown="span">Yes</td>
<td markdown="span">Unique identifier of the span.</td>
<td markdown="span">UUID</td>
</tr>
<tr>
<td markdown="span">`parent`</td>
<td markdown="span">No</td>
<td markdown="span">Identifier of the span's dependent parent, if it has one. This tag is populated as the result of an OpenTracing `ChildOf` relationship.
A span without the `parent` or `followsFrom` tag is the root (first) span of a trace. </td>
<td markdown="span">UUID</td>
</tr>
<tr>
<td markdown="span">`followsFrom`</td>
<td markdown="span">No</td>
<td markdown="span">Identifier of the span's non-dependent parent, if it has one. This tag is populated as the result of an OpenTracing `FollowsFrom` relationship. Wavefront ignores spans with this tag when calculating the critical path through a trace. A span without the `parent` or `followsFrom` tag is the root (first) span of a trace. </td>
<td markdown="span">UUID</td>
</tr>
</tbody>
</table>

The following table lists span tags that describe the architecture of the instrumented application that emitted the span. Wavefront uses these tags to aggregate and filter trace data at different levels of granularity. These tags correspond to the [application tags](tracing_instrumenting_frameworks.html#how-wavefront-uses-application-tags) you set through a Wavefront observability SDK.

<table>
<colgroup>
<col width="20%" />
<col width="10%" />
<col width="55%" />
<col width="15%" />
</colgroup>
<thead>
<tr><th>Span Tags <br>for Filtering</th><th>Required</th><th>Description</th><th>Type</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`application`</td>
<td markdown="span">Yes</td>
<td markdown="span">Name of the instrumented application that emitted the span. </td>
<td markdown="span">String</td>
</tr>
<tr>
<td markdown="span">`service`</td>
<td markdown="span">Yes</td>
<td markdown="span">Name of the instrumented microservice that emitted the span.</td>
<td markdown="span">String</td>
</tr>
<tr>
<td markdown="span">`cluster`</td>
<td markdown="span">Yes</td>
<td markdown="span">Name of a group of related hosts that serves as a cluster or region in which the instrumented application runs. <br>
Specify <strong>cluster=none</strong> to indicate a span that does not use this tag.</td>
<td markdown="span">String</td>
</tr>
<tr>
<td markdown="span">`shard`</td>
<td markdown="span">Yes</td>
<td markdown="span">Name of a subgroup of hosts within the cluster, for example, a mirror.
<br>Specify <strong>shard=none</strong> to indicate a span that does not use this tag.</td>
<td markdown="span">String</td>
</tr>
</tbody>
</table>

Wavefront does not allow the mandatory span tags to have multiple values. Make sure that your application does not send spans with multiple application/service tags.
For example, a span with two span tags `service=notify` and `service=backend` is invalid.

**Note:** Additional span tags may be present, depending on how you instrumented your application. For example, the [framework SDKs](wavefront_sdks.html#sdks-that-instrument-frameworks) automatically use span tags like `component`, `http.method`, and so on. You can find out about these tags in the README file for the SDK on GitHub.

<!---
Because operations are normally composed of other operations, each span is normally related to other spans -  a parent span and children spans.
--->

### Time-Value Precision in Spans

A span has two time-value [fields](#spanfields) for specifying the start time (`start_milliseconds`) and duration (`duration_milliseconds`). Express these values in milliseconds, because Wavefront uses milliseconds for span storage and visualization. For convenience, you can specify time values in other units. Wavefront converts the values to milliseconds.

Wavefront requires that you use the same precision for _both_ time values. Wavefront identifies the precision of the `start_milliseconds` value, and interprets the `duration_milliseconds` value using the same unit. The following table shows how to indicate the start-time precision:

<table>
<colgroup>
<col width="25%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="15%" />
</colgroup>
<thead>
<tr><th>Precision for <br>Start Time Values</th><th>Number Format</th><th>Sample <br>Start Value</th><th>Stored As <br>Milliseconds</th><th>Conversion<br>Method</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">Seconds</td>
<td markdown="span">Fewer than 13 digits</td>
<td markdown="span">`1533529977`</td>
<td markdown="span">`1533529977000`</td>
<td markdown="span">Multiplied by 1000</td>
</tr>
<tr>
<td markdown="span">Milliseconds  <br>(Thousandths of a second)</td>
<td markdown="span">13 to 15 digits</td>
<td markdown="span">`1533529977627`</td>
<td markdown="span">`1533529977627`</td>
<td markdown="span"> -- </td>
</tr>
<tr>
<td markdown="span">Microseconds <br>(Millionths of a second)</td>
<td markdown="span">16 to 18 digits</td>
<td markdown="span">`1533529977627992`</td>
<td markdown="span">`1533529977627`</td>
<td markdown="span">Truncated</td>
</tr>
<tr>
<td markdown="span">Nanoseconds <br>(Billionths of a second)</td>
<td markdown="span">19 or more digits</td>
<td markdown="span">`1533529977627992726`</td>
<td markdown="span">`1533529977627`</td>
<td markdown="span">Truncated</td>
</tr>

</tbody>
</table>

**Note:** When specifying a span in Wavefront span format, make sure you adjust values as necessary so that the units match. For example, suppose you know a span started at `1533529977627` epoch milliseconds, and lasted for `3` seconds. In Wavefront span format, you could specify either of the following pairs of time values:

| `1533529977` | `3` | (both values in seconds) |
| `1533529977627` | `3000` | (both values in milliseconds) |

### Indexed and Unindexed Span Tags

Wavefront uses indexes to optimize the performance of queries that filter on certain span tags. For example, Wavefront indexes the application tags (`application`, `service`, `cluster`, `shard`) so you can quickly query for spans that represent operations from a particular application, service, cluster, or shard. In addition to the application tags, Wavefront indexes certain built-in span tags that conform to the OpenTracing standard, such as `span.kind`, `component`, `http.method`, and `error`.

For performance reasons, Wavefront automatically indexes built-in span tags with low cardinality. (A tag with low cardinality has comparatively few unique values that can be assigned to it.) So, for example, a tag like `spanId` is not indexed.

**Note:** Wavefront does not automatically index any custom span tags that you might have added when you instrumented your application. If you plan to use a low-cardinality custom span tag in queries, contact Wavefront support to request indexing for that span tag.

## Tracing Traffic

Tracing traffic shows how applications and services interact with each other. If you click on a tracing traffic, you can drill down to the trace browser. See [Application Map](tracing_ui_overview.html#application-map-beta) for details.

Each arrow in the image shown below is referred to as a tracing traffic in Wavefront. 
![an image that shows how each service communicates with each other using arrows. These arrows are called tracing traffic in wavefront.](images/tracing_edges_concept.png)

To understand how to query for tracing traffic in the tracing browser, see [Use Spans to Examine Applications and Services](spans_function.html#use-spans-to-examine-applications-and-services).

## RED Metrics

If you instrument your application with a [tracing-system integration](tracing_integrations.html#tracing-system-integrations) or with a [Wavefront OpenTracing SDK](wavefront_sdks.html#sdks-for-collecting-trace-data), Wavefront derives RED metrics from the spans that are sent from the instrumented application. Wavefront automatically aggregates and displays RED metrics for different levels of detail with no additional configuration or instrumentation on your part.

RED metrics are key indicators of the health of your services, and you can use them to help you discover problem traces. RED metrics are measures of:

* Rate of requests – the number of requests being served per minute
* Errors – the number of failed requests per minute
* Duration – per-minute histogram distributions of the amount of time that each request takes


### Span RED Metrics and Trace RED Metrics

Wavefront uses ingested spans to derive RED metrics for two kinds of request:
* Span RED metrics measure individual operations, typically within a single service. For example, a span RED metric might measure the number of calls per minute to the `dispatch` operation in the `delivery` service.

  Wavefront uses span RED metrics as the basis for the [predefined charts](#predefined-charts) shown below.

* Trace RED metrics measure traces that start with a given root operation. For example, a trace RED metric might measure the number of traces that each start with a call to the `orderShirts` operation in the `shopping` service.

  Wavefront derives trace RED metrics from each trace's root span and end span. (If a trace has multiple root spans, the earliest is used.) You need to [query for trace metrics](#red-metrics-queries) to visualize them.

{% include note.html content="For traces that consist entirely of synchronous member spans, trace RED metrics are equivalent to the corresponding span RED metrics. For traces that have asynchronous member spans, trace RED metrics provide more accurate measures of trace duration, especially when a trace's root span ends before a child span." %}


Wavefront automatically generates charts to display the span RED metrics for a particular service. To view these charts, see the [service dashboard](/tracing_ui_overview.html#service-dashboard).

### RED Metric Counters and Histograms

The types of RED metrics that we show in the [predefined charts](#predefined-charts) are rates and 95th percentile distributions. These metrics are themselves based on underlying delta counters and histograms that Wavefront automatically derives from spans. You can use these underlying delta counters and histograms in [RED metrics queries](#red-metrics-queries), for example, to create alerts on trace data.

Wavefront constructs the names of the underlying delta counters and histograms as shown in the table below. The name components `<application>`, `<service>`, and `<operationName>` are string values that Wavefront obtains from the spans on which the metrics are derived. If necessary, Wavefront modifies these strings to comply with the Wavefront [metric name format](wavefront_data_format.html#wavefront-data-format-fields). Wavefront also associates each metric with point tags `application`, `service`, and `operationName`, and assigns the corresponding span tag values to these point tags. The span tag values are used without modification.

{% include warning.html content="Do not configure the Wavefront proxy to add prefixes to metric names. Doing so will change the names of the RED metric counters and histograms, and prevent these metrics from appearing in the Wavefront UI, e.g., in [predefined charts](#predefined-charts)." %}

<table id = "oplevelredmetrics">
<colgroup>
<col width="45%"/>
<col width="15%"/>
<col width="40%"/>
</colgroup>
<thead>
<tr><th markdown="span">Span RED Metric Names</th><th>Metric Type</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`tracing.derived.<application>.<service>.<operationName>.invocation.count`  </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of times that the specified operation is invoked. You can query delta counters using `cs()` queries. <br>Used in the Request Rate chart that is generated for a service. </td>
</tr>
<tr>
<td markdown="span">`tracing.derived.<application>.<service>.<operationName>.error.count`   </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of invoked operations that have errors (i.e., spans with `error=true`). You can query delta counters using `cs()` queries.<br>Used in the Error Rate chart that is generated for a service. </td>
</tr>
<tr>
<td markdown="span">`tracing.derived.<application>.<service>.<operationName>.duration.micros.m`  </td>
<td markdown="span">Wavefront histogram</td>
<td markdown="span">The duration of each invoked operation, in microseconds, aggregated in one-minute intervals. <br>Used in the Duration chart that is generated for a service. </td>
</tr>
</tbody>
</table>


<table id = "tracelevelredmetrics">
<colgroup>
<col width="45%"/>
<col width="15%"/>
<col width="40%"/>
</colgroup>
<thead>
<tr><th>Trace RED Metric Names</th><th>Metric Type</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`tracing.root.derived.<application>.<service>.<operationName>.invocation.count`  </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of traces that start with the specified root operation. You can query delta counters using `cs()` queries.</td>
</tr>
<tr>
<td markdown="span">`tracing.root.derived.<application>.<service>.<operationName>.error.count`   </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of traces that start with the root operation, and contain one or more spans with errors
(i.e., spans with `error=true`). You can query delta counters using `cs()` queries.</td>
</tr>
<tr>
<td markdown="span">`tracing.root.derived.<application>.<service>.<operationName>.duration.millis.m`  </td>
<td markdown="span">Wavefront histogram</td>
<td markdown="span">The duration of each trace, in milliseconds, aggregated in one-minute intervals. Duration is measured from the start of the earliest root span to the end of the last span in a trace.</td>
</tr>
</tbody>
</table>

### RED Metrics Queries

You can perform queries over [RED metric counters and histograms](#red-metric-counters-and-histograms) and visualize the results in your own charts, just as you would do for any other metrics in Wavefront. You can create alerts on trace data by using RED metrics queries in alert conditions.

**Examples**

Find at the per-minute error rate for a specific operation executing on a specific cluster:

```
cs(tracing.derived.beachshirts.shopping.orderShirts.error.count and cluster=us-east-1)
```

Find the per-minute error rate for traces that begin with a specific operation:

```
cs(tracing.root.derived.beachshirts.shopping.orderShirts.error.count)
```

Use a [histogram query](visualize_histograms.html#querying-histogram-metrics) to return durations at the 75th percentile for an operation in a service. (The predefined charts display only the 95th percentile.)

```
percentile(75, hs(tracing.derived.beachshirts.delivery.dispatch.duration.micros.m))
```

**Syntax Alternatives**

Wavefront supports 2 alternatives for specifying the RED metric counters and histograms in a query:
* Use the metric name, for example:
  ```
  cs(tracing.derived.beachshirts.delivery.dispatch.error.count)
  ```
* Use the point tags `application`, `service`, and `operationName` that Wavefront automatically associates with the metric, for example:
  ```
  cs(tracing.derived.*.invocation.count, application="beachshirts" and service="delivery" and operationName="dispatch")
  ```

The point tag technique is useful when the metric name contains string values for `<application>`, `<service>`, and `<operationName>` that have been modified to comply with the Wavefront [metric name format](wavefront_data_format.html#wavefront-data-format-fields). The point tag value always corresponds exactly to the span tag values.

### Aggregated RED Metrics

Wavefront computes service level RED metrics by aggregating the [RED metrics derived from spans](#span-red-metrics-and-trace-red-metrics). Querying and aggregating these metrics can be slow due to high cardinality from operation tags, source tags, and custom tags. Therefore, to make RED metric querying faster, wavefront introduced pre-aggregated RED metrics.

Wavefront constructs the names of the underlying aggregated delta counters, and histograms as shown in the table below. The `<application>` and `<service>` in the name are string values that Wavefront obtains from the spans on which the metrics are derived. You can filter the aggregated RED metrics using the `application`, `service`, `cluster`, `shard`, `source`, and `span.kind` point tags. Wavefront assigns the corresponding span tag values to these point tags.

{% include note.html content="If you want to filter RED metrics using operation names, sources, or [custom span tags](tracing_customize_spans_and_alerts.html), use [RED metrics derived from spans](#span-red-metrics-and-trace-red-metrics)." %}

![the screenshot shows the filters available for an aggregated red metrics query in the chart builder.](images/tracing_aggregated_red_metrics.png)

<table id = "spanaggregatedREDmetrics">
<colgroup>
<col width="45%"/>
<col width="15%"/>
<col width="40%"/>
</colgroup>
<thead>
<tr><th>Aggregated RED Metric Names</th><th>Metric Type</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`tracing.aggregated.derived.<application>.<service>.invocation.count`  </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of spans for the specified application and service.</td>
</tr>
<tr>
<td markdown="span">`tracing.aggregated.derived.<application>.<service>.error.count`   </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of spans that have errors for the specified application and service.
(i.e., spans with `error=true`).</td>
</tr>
<tr>
<td markdown="span">`tracing.aggregated.derived.<application>.<service>.duration.millis.m`  </td>
<td markdown="span">Wavefront histogram</td>
<td markdown="span">The duration of each spans, in microseconds, aggregated in one-minute intervals. Duration is measured from the start of the earliest root span to the end of the last span in a trace.</td>
</tr>
</tbody>
</table>

**Examples**

* Request rate or invocation count for an edge: Find the per-minute request rate for a specific application.
  ```
  cs(tracing.aggregated.derived.beachshirts.*.invocation.count)
  ```
  
* Error percentage for an edge: Find the per-minute aggregated error rate for traces for a specific `span.kind` tag.
  ```
  cs(tracing.aggregated.derived.beachshirts.shopping.error.count, span.kind=server)
  ```
  
* Duration in the form of Wavefront histograms: Find the 95th percentile of a specific service using aggregated RED metrics.
  ```
  alignedSummary(95, merge(hs(tracing.aggregated.derived.beachshirts.delivery.duration.micros.m)))
  ```

### RED Metrics for Tracing Traffic
 
You can visualize tracing traffic data in charts using tracing traffic derived metrics and filter them using the point tags listed below. Wavefront assigns the corresponding span tag values to these point tags. The span tag values are used without modification.

![the screenshot shows the filters available for a traffic derived red metrics query in the chart builder. The filers are explained after the screenshot.](images/edge_derived_red_metrics.png)

<table>
<colgroup>
<col width="20%"/>
<col width="80%"/>
</colgroup>
<tr>
  <th>
    Point Tags
  </th>
  <th>
    Description
  </th>
</tr>
<tr>
  <td>
  <code>application</code>
  </td>
  <td>
  Name of the application the request is sent from.
  </td>
</tr>
<tr>
  <td>
    <code>service</code>
  </td>
  <td>
  Name of the microservice that request is sent from.
  </td>
</tr>
<tr>
  <td>
  <code>cluster</code>
  </td>
  <td>
  Name of a group of related hosts that serves as a cluster or region in which the application that sent the request runs.
  </td>
</tr>
<tr>
  <td>
  <code>shard</code>
  </td>
  <td>
  Name of a subgroup of hosts within the cluster that sent the request, for example, a mirror.
  </td>
</tr>
<tr>
  <td>
  <code>source</code>
  </td>
  <td>
  Name of a host or container on which the applications or services sent requests.
  </td>
</tr>
<tr>
  <td>
  <code>to.application</code>
  </td>
  <td>
  Name of the application the request is sent to.
  </td>
</tr>
<tr>
  <td>
  <code>to.service</code>
  </td>
  <td>
  Name of the service the request is sent to.
  </td>
</tr>
<tr>
  <td>
  <code>to.cluster</code>
  </td>
  <td>
  Name of a group of related hosts that serves as a cluster or region in which the application that received the request runs.
  </td>
</tr>
<tr>
  <td>
  <code>to.shard</code>
  </td>
  <td>
  Name of a subgroup of hosts within the cluster that received the request, for example, a mirror.
  </td>
</tr>
<tr>
  <td>
  <code>to.source</code>
  </td>
  <td>
  Name of a host or container on which the applications or services received requests.
  </td>
</tr>
</table>

<table id = "edgederivedREDmetrics">
<colgroup>
<col width="45%"/>
<col width="15%"/>
<col width="40%"/>
</colgroup>
<thead>
<tr><th>Tracing Traffic Derived Metric Name</th><th>Metric Type</th><th>Description</th></tr>
</thead>
<tbody>
<tr>
<td markdown="span">`tracing.edge.derived.<application>.<service>.invocation.count`  </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of tracing traffic that start from an application and service.</td>
</tr>
<tr>
<td markdown="span">`tracing.edge.derived.<application>.<service>.error.count`   </td>
<td markdown="span">Delta counter</td>
<td markdown="span">The number of tracing traffic that starts from an application and service, that are errors.</td>
</tr>
<tr>
<td markdown="span">`tracing.edge.derived.<application>.<service>.duration.millis.m`  </td>
<td markdown="span">Wavefront histogram</td>
<td markdown="span">The duration of the request in milliseconds, aggregated in one-minute intervals.</td>
</tr>
</tbody>
</table>

**Examples**

* Request rate or invocation count for a tracing traffic.
  ```
  cs(tracing.edge.derived.beachshirts.*.invocation.count, to.shard=primary)
  ```
* Error percentage for a tracing traffic. 
  ```
  cs(tracing.edge.derived.beachshirts.shopping.error.count, cluster=us-west)
  ```
* Duration in the form of Wavefront histograms.
  ```
  hs(tracing.edge.derived.beachshirts.shopping.duration.millis.m, to.service=delivery)
  ```

### Trace Sampling and Derived RED Metrics

If you have instrumented your application with a Wavefront observability SDK, Wavefront derives the RED metrics from 100% of the generated spans, _before_ any sampling is performed. This is true when the sampling is performed by the SDK or when the sampling is performed by a Wavefront proxy. Consequently, the RED metrics provide a highly accurate picture of your application's behavior. However, if you click through a chart to inspect a particular trace, you might discover that the trace has not actually been ingested in Wavefront. You can consider configuring a less restrictive [sampling strategy](trace_data_sampling.html).

If you have instrumented your application using a 3rd party distributed tracing system, Wavefront derives the RED metrics _after_ sampling has occurred. The Wavefront proxy receives only a subset of the generated spans, and the derived RED metrics will reflect just that subset. See [Trace Sampling and RED Metrics from an Integration](tracing_integrations.html#trace-sampling-and-red-metrics-from-an-integration).

## Application Tags

An ApplicationTags object  describes your application to Wavefront. Wavefront requires tags that describe the structure of your application. These application tags are associated with the metrics and trace data that the instrumented microservices in your application send to Wavefront.

Application tags and their values are encapsulated in an `ApplicationTags` object in your microservice's code. You specify a separate `ApplicationTags` object, with a separate set of tag values, for each microservice you instrument. The tags include information about the way your application is structured and deployed, so your code normally obtains tag values from a configuration file at runtime. The configuration file might be provided by the Wavefront SDK, or it might be part of a custom configuration mechanism that is implemented by your application. (Only SDKs with quickstart setup steps provide a configuration file.)

{% include note.html content="You can use an `ApplicationTags` object to store any additional custom tags that you want to associate with reported metrics or trace data." %}

### How Wavefront Uses Application Tags

Wavefront uses application tags to aggregate and filter data at different levels of granularity.

* **Required tags** enable you to drill down into the data for a particular service:
    - `application` - Name that identifies the application, for example, `beachshirts`. All microservices in the same application should use the same `application` name.
    - `service` - Name that identifies the microservice, for example, `shopping`. Each microservice should have its own `service` name.

  ![tracing app services](images/tracing_app_services_page.png)


* **Optional tags** enable you to use the physical topology of your application to further filter your data:
  - `cluster` - Name of a group of related hosts that serves as a cluster or region in which the application will run, for example, `us-west`.
  - `shard` - Name of a mirror or other subgroup of hosts within a cluster, for example, `primary`.

  ![tracing service filter](images/tracing_service_filter_page.png)

## Span Logs

The OpenTracing standard supports [span logs](https://opentracing.io/docs/overview/spans/#logs). You can use a Wavefront SDK to instrument your application to include span log information.

{% include note.html content="Span logs are disabled by default and require Wavefront proxy version 5.0 or later. Contact [support@wavefront.com](mailto:support@wavefront.com) to enable the feature." %}

You can instrument your application to emit one or more logs with a span, and examine the logs from the Tracing UI. For details on how to add a `log()` method for a specific SDK, see the OpenTracing SDK.

Here's an example that adds span logs to [the best practices example](tracing_best_practices.html#best-practices-for-wavefront-observability-sdks-3) to emit a span log in case of an exception:

![span log example](images/span_log_example.png)

Span logs are especially useful for recording additional information about errors within the span.


## Helper Objects That Collect and Transfer Data
The actual helper objects in a microservice depends on the SDKs you set up. A typical set of helper objects includes some or all of the following:
{% include note.html content="When you use multiple Wavefront SDKs to instrument a microservice, certain helper objects belong to exactly one SDK, and other helper objects are shared."%}

### Wavefront Sender

When you instrument an application, you set up a mechanism for sending metrics and trace data to the Wavefront service, as described in [Step 1, Prepare to Send Metrics to Wavefront,](#step-1-prepare-to-send-data-to-wavefront) above. Choose between:

* Sending data directly to the Wavefront service, also called [direct ingestion](direct_ingestion.html).
* Sending data to a [Wavefront proxy](proxies.html), which then forwards the data to the Wavefront service.

Your choice is represented in your code as Wavefront Sender object.
(Most Wavefront SDKs define objects of type `WavefrontSender` or simply `Sender`. A few SDKs define a pair of separate `Client` objects.) A Wavefront sender encapsulates the settings you supply when you instrument your microservice. The settings in your code must match the information you provided in [Step 1](#step-1-prepare-to-send-data-to-wavefront) above.

{% include note.html content="You can use a Wavefront sender to tune performance by setting the frequency for flushing data to the Wavefront proxy or the Wavefront service. If you are using direct ingestion, you can also change the defaults for batching up the data to be sent." %}

<!--- change links when proxy/dir ing decision is in a single section --->

### WavefrontTracer and WavefrontSpanReporter

Wavefront uses a pair of objects to create and report trace data:

* A `WavefrontTracer` creates spans and traces.
* A `WavefrontSpanReporter` forwards the trace data to the Wavefront sender.

A `WavefrontSpanReporter` specifies the source of the reported trace data -- by default, the host that the code is running on. You can optionally specify a more useful source name explicitly during setup, for example, an IP address, a container or instance name, or some other unique data source. All reporter objects for a particular microservice must specify the same source.

Trace data is reported automatically whenever spans are complete, so a `WavefrontSpanReporter` does not specify a reporting interval.

{% include note.html content="If you need to debug issues with spans, you can set up a `CompositeReporter` to combine a `WavefrontSpanReporter` with a `ConsoleReporter`. A `ConsoleReporter` sends trace data to your console." %}

### Wavefront Metrics Reporter Objects

Wavefront uses one or more reporter objects to gather metrics and histograms and forward that data to the Wavefront sender. Different Wavefront reporter objects gather data from different components of your application. For example, a `WavefrontJvmReporter` reports runtime data from the JVM.

A Wavefront reporter object specifies:
* The reporting interval for metrics and histograms. The reporting interval controls how often data is reported to the Wavefront sender and therefore determines the timestamps of data points sent to Wavefront. The default reporting interval is once a minute.

* The source of the reported metrics and histograms -- by default, the host that the code is running on. You can optionally specify a more useful source name explicitly during setup, for example, an IP address, a container or instance name, or some other unique data source. All reporter objects for a particular microservice must specify the same source.

{% include note.html content="You can use a Wavefront reporter object to set a nondefault reporting interval." %}
