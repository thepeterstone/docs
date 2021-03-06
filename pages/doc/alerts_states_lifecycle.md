---
title: Alert States and Lifecycle
tags: [alerts, videos]
sidebar: doc_sidebar
permalink: alerts_states_lifecycle.html
summary: Learn about alert conditions and states, when alerts fire, and how alerts resolve.
---
Here's a video to get you started:

<p><a href="https://vmwarelearningzone.vmware.com/oltpublish/site/openlearn.do?dispatch=previewLesson&id=6cb2ac52-dc7a-11e7-a6ac-0cc47a352510&inner=true&player2=true"><img src="/images/v_alerts_lifecycle.png" style="width: 700px;"/></a>
</p>

## Alert Lifecycle Basics

The alert lifecycle determines which events the alert triggers, and which alert targets get notifications. Classic and multi-threshold alerts are exactly the same in terms of events. However, when notifications are sent to which target differs for classic alerts and for multi-threshold alerts.

### When Classic Alerts Notify Targets

Classic alerts notify all targets at the same time when the alert changes state:
1. Wavefront monitors the alert condition. When the condition is met for the specified amount of time, the alert fires.
2. When the alert fires, Wavefront sends alert notifications to the alert target(s) specified for the alert, using the severity that's prespecified for the alert.
3. When the alert resolves or is snoozed, Wavefront sends additional notification to the alert target(s).

### When Multi-Threshold Alerts Notify Targets

A multi-threshold alert supports multiple severities with different alert targets. Let's look at an example:

|ts expression |`ts(cpu.loadavg.1m)`|
|operator | >|
|SEVERE  | 6000  |
|WARN   | 5000  |
|SMOKE  | 4000  |

This multi-threshold alert notifies targets like this:
1. Wavefront monitors the alert condition.
2. If at least one of the threshold values is met for the specified amount of time, for example, if `cpu.loadavg.1m` is greater than 6000, the alert fires.
3. Notifications are always sent to all alert targets that match the condition, for example, that are equal to or below the severity that triggers the alert. For example, if `cpu.loadavg.1m` is greater than 6000 for 5 minutes, alert targets for SEVERE, WARN, and SMOKE are notified because the condition is satisfied for all. If the value of `cpu.loadavg.1m` satisfies the WARN but not the SEVERE condition, then only alert targets in WARN and SMOKE will be notified.
4. Wavefront continues checking the alert condition at the specified interval (1 minute by default). If the alert condition for a higher level is no longer met, but lower-level conditions are still met, then the higher-level alert target gets an Alert Resolved notification, and each lower-level alert target gets an Alert Updated notification.

![alert multi concept](images/alert_multi_concept.png)

## Alert States

An alert can be in one of the following states:

<table id="alert-properties">
<tbody>
<thead>
<tr><th width="20%">Alert State</th><th width="80%">Description</th></tr>
</thead>
<tr>
<td><strong>CHECKING</strong></td>
<td>We check whether the <strong>Condition</strong> is met for the amount of time specified by the <strong>Alert fires</strong> property.
Alerts can't be in the CHECKING and the FIRING state at the same time even though firing alerts are being checked to determine if firing conditions are still being met.  An alert resolves and transitions back to CHECKING when no true values are present within the time window, or when the time window contains no data.</td></tr>
<tr>
<td><strong>FIRING</strong></td>
<td>The alert is meeting the <strong>Condition</strong> for the amount of time specified by the <strong>Alert fires</strong> property. An alert transitions to FIRING when the condition evaluates to at least one true value and no false values during a fixed time window.</td>
</tr>
<tr>
<td><strong>NO DATA</strong></td>
<td markdown="span">The series for which an alert is defined is not reporting data.
You can set up an alert that triggers if the alert is in a NO DATA state for a specified amount of time. If you do, select **Alert Has No Data** in the corresponding [Alert Target](/webhooks_alert_notification.html#creating-an-alert-target). For those alert targets, select **Alert Has No Data Resolved** if you want to send a notification when the alert exits the NO DATA state.</td></tr>
<tr>
<td><strong>IN MAINTENANCE</strong></td>
<td>The alert has an alert tag or a source or set of sources included in a source tag associated with an ongoing maintenance window.
<ul><li>If an alert has a subset of reporting sources associated with in an ongoing maintenance window, then the state displays as CHECKING/IN MAINTENANCE.</li>
<li>If an alert has a subset of reporting sources associated with an ongoing maintenance window but the other sources are firing, the state displays as FIRING/IN MAINTENANCE.</li></ul>
 </td>
</tr>
<tr>
<td><strong>INVALID</strong></td>
<td>The alert is timing out ( > 5 min query execution) or queries include inactive metrics or sources. When an alert is in the INVALID state, it is checked approximately every 15 minutes, instead of the specified checking frequency (see next section).</td></tr>
<tr>
<td><strong>SNOOZED</strong></td>
<td>The alert is not checked.</td></tr>
</tbody>
</table>


## When Alerts Are Checked

The time series associated with an alert are checked to determine whether the alert should fire or not. The default checking frequency interval is 1 minute. This means that the conditional expression associated with the alert is evaluated once a minute. You can change this interval by setting the **Checking Frequency** advanced property when you create or edit the alert.

The exact time of the check for a particular alert is not fixed and can vary slightly within the minute. For example, for a specific alert it’s possible that a check occurs at 1:01:17pm and the next check occurs at 1:02:13pm.

## Data Granularity for Alert Checking

The data granularity for alert checking is 1 minute. The alert checking process:

1. Evaluates the ts() expression you specified in the alert condition.
1. Implicitly aligns the returned values by grouping them into 1-minute buckets.
1. Summarizes the values within each bucket by averaging them, and aligns each result at the beginning of the minute that contains the summarized values.
1. Evaluates each average value as true (non-zero) or false (zero).

If the ts() expression returns a single data value per minute, the summarization values and the returned values are the same.

{% include note.html content="If you want a different summarization strategy, then you can explicitly use the [`align()`](ts_align.html) function in your alert condition, with parameters specifying a 1-minute time window and your preferred summarization method." %} 

**Example 1**

Suppose your alert condition is `ts(my.metric) > 4` and `my.metric` reports 5 data values (9,9,9,3,9) between 12:11:00pm and 12:11:59pm. The alert checking process:
1. Compares each reported value to the condition
2. Returns a 1 (true) or 0 (false) for each value
3. Produces a single summarization data point at 12:11:00pm.

The value of this summarization point is .8:

`(((1+1+1+0+1) / 5) = .8)`

This value (.8) evaluates to true because .8 != 0.

**Example 2**

Suppose you want to return an entire summarized minute value as false if there's at least 1 false value present. You can explicitly bucket the values by changing your alert condition to use `align()`:

`align(1m, min, ts(my.metric) > 4)`.

When my.metric reports the data values (9, 9, 9, 3, 9) between 12:11:00pm and 12:11:59pm, the alert checking process:
1. Compares each value to the condition (>4)
2. Returns a true (1) or false (0) value.

The `align()` function returns the minimum value (0) as the single value at 12:11:00pm. For that summarized minute value, the alert checking process returns false because 0 = 0.

## Alert Check Time Window

The time window that we evaluate at each checking frequency interval depends on the state of the alert:

- When an alert is currently not firing, the **Alert fires** property determines the time window that is evaluated.
- When an alert is currently firing, the **Alert resolves** property determines the time window that is evaluated.

The data points that are evaluated during a check time window are the [1-minute summarizations](#data-granularity-for-alert-checking) described above.
E.g., if the **Alert fires** property is set to 3 minutes, then the alert check evaluates 3 summarization data points, one for each minute in the check time window.

The last summarization data point to be evaluated in an alert check time window is determined by the following formula:

 ```
 alert check time (rounded down to nearest minute) - 1 minute
 ```

We use this formula to ensure that the alert has a full minute's worth of reported data to summarize and evaluate.

**Example**

Suppose the **Alert fires** property is set to 5 minutes, and the alert check time is 1:09:32pm:
* The last summarization data point to be evaluated is at 1:08:00pm `((1:09:32 - 0:00:32) - 0:01:00)`. This point is the average of the data values that were returned by the alert query from 1:08:00pm to 1:08:59pm.
* The 5-minute time window includes the 5 summarization data points from 1:04 - 1:08. These points cover the data values returned from 1:04:00pm through 1:08:59pm.

## When Alerts Fire

An alert fires when its [condition](#alert-conditions) evaluates to at least one true value and zero false values present within the given **Alert fires** time window.

**Example**

Suppose the alert condition is `ts(my.metric) > 4` and the **Alert fires** window is 2 minutes:
- If the metric reports exactly one data value (5) in the last 2 minutes, and no other points (no data), the alert fires. <br> Reason: The **Alert fires** window contains one true summarization value (5 > 4) and no false values.
- If the metric reports many points in the last 2 minutes, all of which are <= 4, the alert does not fire. <br> Reason: The **Alert fires** window contains two false summarization values, because the averages of values that are <= 4 are also <= 4.
- If the metric reports many points in the last 2 minutes, all of which are > 4, the alert fires. <br> Reason: The **Alert fires** window contains two true summarization values and no false values, because the averages of values that are > 4 are also > 4.
- If the metric reports exactly two data values (5 and 3), anywhere in the last 2 minutes, the alert does not fire. <br> Reason: The **Alert fires** window contains one false summarization value. If each data value is in a different minute, then one of the summarization values is 3, which evaluates to false. If both data values are in the same minute, then their average (4) evaluates to false (4 > 4 is false).

Alert checks are based on data that is summarized every minute.  Consequently, if `ts(my.metric)` returns 5, 5, and 3 in the same minute, the summarized value (4.33) evaluates to true for that minute because 4.33 > 4, even though 3 by itself evaluates to false. All alert queries are checked according to the **Checking Frequency** property.


## Viewing Firing Alerts

The alerts icon in the task bar ![number of alerts](images/alerts.png#inline) shows the number of alerts firing and their severity. The filter bar at the left of the Alerts page shows the number of firing alerts by severity. For multi-threshold alerts, we list each alert only for the highest severity even if lower severity conditions are also met.

You can click the FIRING facet to filter the list of alerts:

![Tag path](images/alerts_filter.png)


## When Alerts Resolve

An alert resolves when there are either no true values present within the given **Alert resolves** time window, or when the **Alert resolves** time window contains no data. By default, the **Alert resolves** time window is the same length as the **Alert fires** time window.

**Example**

Suppose you define an alert with the following properties:
* The alert condition is `ts(metric.name) > 0`, where `metric.name` reports once a minute. (The summarization values are therefore the same as the reported values.)
* The [Checking Frequency interval](#when-alerts-are-checked) = 1 minute (the default).
* **Alert fires** = 5 minutes.
* **Alert resolves** = 10 minutes.

Here's how the alert might fire, and then resolve:

 1. `metric.name` normally reports 0, but starts reporting 1 at 10:21.
 1. At 10:26, the alert fires, because `metric.name` has reported 1 during the 5 whole minutes (from 10:21 to 10:25) immediately before the alert check at 10:26.
 1. At 10:26, `metric.name` starts reporting 0 again. It stops reporting for a few minutes of no data, and then continues reporting 0.
 1. At 10:36, the alert resolves, because `metric.name` has reported 0 during the 10 whole minutes (from 10:26 to 10:35) immediately before the alert check at 10:36.

![alerts_basic_fire_resolve](images/alerts_basic_fire_resolve.png)

## Alert Lifecycle Example

Suppose the threshold for an alert is set to 50%, and
the alert's settings are **Alert fires** = 2 minutes, **Alert resolves** = 2 minutes, and **Checking Frequency** = 1 minute.

In the chart below:
* An event window from 09:34 to 09:35 identifies the interval during which the metric crossed the threshold going up.
* An event window from 09:39 to 09:40 identifies the interval during which the metric crossed the threshold going down.
* The alert fires around 09:37:09 and resolves at 09:41:59.

![Alert fires](images/alert_fire.png)

Why does the alert fire when it does?
* An alert check occurs at 09:37:09, and takes into account the 2 summarization data points at 09:35 and 09:36. Each summarization data point evaluates to true, because it is the average of values that are all > 50%.
* The alert fires because the alert check finds two true values and no false values among the summarization data points in the time window.

Why does the alert resolve when it does?
* An alert check occurs at 09:41:59, and takes into account the 2 summarization data points at 09:39 and 09:40. Each summarization data point evaluates to false:
  - The summarization point at 09:40 is the average of values (all <= 50%) that were reported from 09:40 to 09:40:59.
  - The summarization point at 09:39 is the average of the values (some > 50%, some <= 50%) that were reported from 09:39 to 09:39:59. The resulting average is 44%, which makes the summarization value false (44% <= 50%).
* The alert resolves because the alert check finds at no true values among the summarization data points in the time window.

## Did My Alert Misfire?

The alert checking process bases its decisions on the values that are actually present at the time of the alert check. If all metrics report their data points on time, then alert checking bases its decisions on a complete picture of your metrics. Sometimes, however, the alert checking process must evaluate temporarily incomplete data when deciding whether or not to fire or resolve. The resulting alert decision might produce:
* An apparent false positive, e.g., an alert that fires, but later looks like it shouldn’t have.
* An apparent false  negative, e.g., an alert doesn't fire, but later looks like it should have.

If you suspect an apparent false positive or negative, you can:
* Check for [delayed data reporting](alerts_delayed_data.html#check-for-a-data-delay).
* [Adjust your alert condition](#minimize-the-impact-of-data-delays-on-alerts) to prevent the alert from responding until data reporting is complete.



<!---  combine this with best practices

### Picking Alert Time Window Values

You pick time window values according to your use case:
* For a fast fire, slow-resolve alert, you can set **Alert fires** < **Alert resolves**.
* For a slow-fire, fast-resolve alert, you can set **Alert fires** > **Alert resolves**.
*

When **Alert fires** > **Alert resolves**, Wavefront adjusts the firing rules to require at least one true value during the **Alert resolves** window before the alert can fire again. This adjustment prevents successive **Alert fires** windows from overlapping with previous ones, which results in unwanted firings immediately after a resolve window in which no data is reported.
--->

<!---
- An aggregate function is used in the alert query and missing data was present for one or more underlying series at the time the alert fired. This tends to make up the majority of misfiring alerts. If there is at least one truly reported data value present at a given time window for one of the underlying series, then Wavefront attempts to apply an interpolated value for all underlying series that did not report a value at that given moment in time. For example, suppose you are aggregating data for `app-1`, `app-2`, and `app-3` using the `sum()` aggregate function. `app-1` and `app-3` reported values at 1:00:00pm and 1:03:00pm, while `app-2` reported values at 1:00:00pm, 1:01:00pm, and 1:03:00pm. In this case, an interpolated value is applied for `app-1` and `app-3` at 1:01:00pm because `app-2` reported a value at that moment in time.

  Now assume that the end of the alerting check time window is 1:02:00pm. To apply accurate interpolated values, a reported value must exist before and after the moment of interpolation. Because `app-1` and `app-3` don't report a value until 1:03:00pm, it's impossible to interpolate a value for them at 1:02:00pm. At 1:03:00pm, the data values for `app-1` and `app-3` are reported and therefore interpolated values are retroactively applied to 1:02:00pm for these sources. If the alerting check evaluates the data before the interpolated values are applied, then it's possible that the interactive chart you view 5 or 10 minutes later does not show the value that the alerting check originally saw.

--->
