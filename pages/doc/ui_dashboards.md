---
title: Create and Customize Dashboards
tags: [getting started, dashboards, charts]
sidebar: doc_sidebar
permalink: ui_dashboards.html
summary: Create dashboards, add charts, and customize dashboard layout.
---

<table style="width: 100%;">
<tbody>
<tr>
<td width="80%">Wavefront dashboards allow you organize and customize the information about your environment. For example:
<ul>
<li>Organize charts into sections</li>
<li>Perform global operations such as setting the dashboard time window.</li>
<li>Use dashboard variables.</li></ul></td>
<td width="20%"><a href="ui_dashboards_v1.html"><img src="/images/classic_button.png" alt="click here for the classic doc"/></a></td>
</tr>
</tbody>
</table>

[Examine Data with Dashboards and Charts](ui_examine_data.html) explains how to set dashboard preferences, set the dashboard time window, isolate sources and series, and more.

{% include shared/badge.html content="Every Wavefront user can view dashboards and make some changes such as setting the time window. You must have Dashboard permission and Modify access to save changes you make to dashboards." %}

## Video

Users with Dashboards permissions can create a new dashboard with one or multiple charts from metrics, a chart type, or an integration.

<p>
<iframe src="https://bcove.video/2WxBJoe" width="700" height="400" allowfullscreen="true" alt="creating dashboards video"></iframe>
</p>


## Create a Dashboard

You have several options for creating a dashboard:

* Select **Dashboards > Create Dashboard**, drag in the Metrics or New Chart widget, and follow the wizard to create a single-chart or multi-chart dashboard.
* Select **Dashboards > Create Dashboard**, drag in the Templates widget, and select an integration, then pick the dashboards and charts you'd like to include.
* Select **Dashboards > All Dashboards** and click **Create Dashboard**
* Select **Browse > Metrics** and click **Create Dashboard**.

### Create a Dashboard from Metrics or Charts

It's easy to create a dashboard from metrics or by selecting a chart.

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<strong>To create a dashboard</strong>:
<ol><li>Select <strong>Dashboards > Create Dashboard</strong> from the task bar. </li>
<li>Drag the <strong>Metrics</strong> or <strong>New Chart</strong> widget onto the canvas</li>
<li>Select metrics, filters, and functions now or later. </li>
<li>In the top right, click <strong>Save</strong> and specify a name and URL for the dashboard. The URL field supports letters, numbers, underscores, and dashes.  The Name field supports letters, numbers, characters, and spaces.</li></ol></td>
<td width="60%"><img src="/images/v2_create_dashboard.png" alt="create dashboard"></td>
</tr>
</tbody>
</table>

### Create a Dashboard from Integration Templates

With release 2019.46, you can create a dashboard by specifying an integration dashboard as a template.

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<strong>To create a dashboard</strong>:
<ol><li>Select <strong>Dashboards > Create Dashboard</strong> from the task bar. </li>
<li>Drag the <strong>Templates</strong> widget onto the canvas. </li>
<li>Select first the source integration, then the dashboard you want as a template, and then one or more charts from that dashboard.</li>
<li>In the top right, click <strong>Save</strong> and specify a name and URL for the dashboard. The URL field supports letters, numbers, underscores, and dashes.  The Name field supports letters, numbers, characters, and spaces.</li></ol></td>
<td width="60%"><img src="/images/v2_create_dashboard_template.png" alt="create dashboard from template"></td>
</tr>
</tbody>
</table>

### Create a Dashboard from a Tracing Template

The Wavefront service dashboard includes a set of charts to monitor the trace data sent by each service in your application. You can use the service dashboard as a template to create your dashboard and then customize the charts for your environment.

{% include note.html content="To view data in these charts, your applications need to send trace data to Wavefront. See [Instrument Your Applications](tracing_instrumenting_frameworks.html) for details." %}

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<strong>To create a dashboard</strong>:
<ol><li>Select <strong>Dashboards > Create Dashboard</strong> from the taskbar. </li>
<li>Drag the <strong>Tracing Templates</strong> widget onto the canvas. </li>
<li>Click <strong>Import Charts</strong>.</li>
<li>In the top right, click <strong>Save</strong> and specify a name and URL for the dashboard.
  <ul>
  <li>The URL field supports letters, numbers, underscores, and dashes.</li>
  <li>The Name field supports letters, numbers, characters, and spaces.</li>
  </ul>
</li>
<li>To view data that is specific to an application and service, use the <strong>application</strong> and <strong>service</strong> dropdowns.</li>
</ol></td>
<td width="60%"><img src="/images/create_tracing_template.png" alt="create dashboard from template"></td>
</tr>
</tbody>
</table>

**Take a look at the cool actions you can do using these charts:**
* [Navigate to the Tracing UI from the Service Dashboard](tracing_ui_overview.html#navigate-to-the-tracing-ui-from-the-service-metrics-dashboard).
* [Explore the Default Service Dashboard](tracing_ui_overview.html#explore-the-default-service-dashboard).
* To delete or edit a chart, see [Clone, Delete, or Edit a Chart](#clone-delete-or-edit-a-chart).

## Edit or Clone a Dashboard

The dashboard menu allows you to create a dashboard, edit a dashboard, clone a dashboard, and look at the dashboard version history.

* When you **clone** a dashboard, you copy the dashboard. If you want to customize one of the Wavefront read-only dashboards, such as integration dashboards, just clone the dashboard and edit the clone.
* The dashboard  **version history** tracks each saved version and includes the user who saved the version. The result is an audit trail for the dashboard.

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<strong>To edit a dashboard</strong>:
<ol><li>Click the ellipsis icon in the top right of the dashboard. </li>
<li>Select <strong>Edit</strong> and make changes to the dashboard in edit mode.</li>
<li>Save the dashboard.</li></ol></td>
<td width="60%"><img src="/images/v2_edit_dashboard.png" alt="edit a dashboard"></td>
</tr>
<tr>
<td width="40%">
<strong>To clone a dashboard</strong>:
<ol><li>Click the ellipsis icon in the top right of the dashboard and select <strong>Clone</strong>. </li>
<li>Accept the suggested URL and dashboard name, or specify a new URL and dashboard name. Specify only the URL string. Do not include <code>https://</code>.  </li>
<li>Save the cloned dashboard.</li></ol></td>
<td width="60%"><img src="/images/v2_clone_dashboard.png" alt="clone a dashboard"></td>
</tr>
</tbody>
</table>

## Examine Metrics in Dashboard View Mode

All users can examine metrics, set the time window, and make temporary changes to dashboards. See [Examine Data](ui_examine_data.html) for details.

![dashboard elements](images/v2_dashboard_elements.png)

Here are some examples of what [all users can do](ui_examine_data.html):
* Set the dashboard time window
* Find a dashboard section
* Filter with global filters or dashboard variables
* Find a dashboard
* Isolate sources or series
* Fine-tune the time window


## Make Changes to a Dashboard in Edit Mode

When you create a dashboard or when you edit a dashboard, the dashboard is in Edit mode. In Edit mode, you can make several changes at a time, then save all changes to dashboard layout or to charts.

{% include shared/system_dashboard.html %}

<!---
To remove a change, click the revert icon to the left of **Edit JSON** on the task bar. The revert icon removes changes starting with the most recent and works backward. You can remove only changes made in the current Edit mode session.--->

![dashboard in edit mode](images/v2_dashboard_edit.png)

### Add a Chart Using Drag and Drop

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">
<ol><li>Drag and drop widgets onto the dashboard canvas </li>
<li>(Optional) Select metrics, filters, and functions.  </li>
<li>Scroll up and select <strong>Save</strong></li>
</ol></td>
<td width="50%"><img src="/images/v2_add_chart_wizard.png" alt="add chart wizard"></td>
</tr>
</tbody>
</table>

### Clone, Delete, or Edit a Chart

Editing a chart is different in View mode and in Edit mode:
* With a dashboard in View mode, click the chart title to edit a chart. <br>
* With a dashboard in Edit mode, use the icons on any chart.

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">
<ol><li>Place the cursor inside a chart. </li>
<li>Click one of the icons and follow the prompts. </li>
</ol>
</td>
<td width="50%"><img src="/images/v2_dashboard_edit_chart.png" alt="clone a chart"></td>
</tr>
</tbody>
</table>

### Add or Edit Dashboard Variables

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">
With the dashboard in Edit mode:
<ol>
<li>Scroll up to the Variables bar.  </li>
<li>Click the Edit icon to edit a variable.</li>
<li>Click the Add button to <a href="dashboards_variables.html">add a variable</a>.</li>
</ol>
</td>
<td width="50%">
<img src="images/v2_dashboard_variables.png" alt="select variables"></td>
</tr>
</tbody>
</table>

### Change Dashboard Layout

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">With the dashboard in Edit mode:
<ul><li>Rearrange charts within section or across sections (12 charts per row maximum). The grid determines what's possible.</li>
<li>Click the canvas to add a section. </li>
<li>Delete or move a section using the icons on the right of the section bar.</li>
</ul>
</td>
<td width="50%"><img src="/images/v2_add_section.png" alt="add section"></td>
</tr>
</tbody>
</table>


### Add a New or Cloned Chart

When you create a chart using **Dashboards > Create Chart**, you're prompted to save it to a dashboard. When you edit a chart, you can save to the current dashboard, or save a clone to a different dashboard.

<table style="width: 100%;">
<tbody>
<tr>
<td width="50%">
<ol><li>With the dashboard in View mode, create or edit a chart. </li>
<li>Select the <strong>v</strong> next to <strong>Save</strong> and make a selection:
<ul><li>To save to an existing dasbhoard, start typing the name of the dashboard, select a dashboard, and click <strong>Insert</strong></li>
<li>Click <strong>Save to New Dashboard</strong>, enter the dashboard name and URL, and click <strong>Create</strong>. Specify only the URL string; do not include https://. </li> </ul></li>
<li>When the target dashboard opens in Edit mode, click and drag the chart to the location of your choice and click <strong>Save</strong> at the top.</li></ol></td>
<td width="50%">
<img src="/images/v2_save_chart_to_new.png " alt="save to dashboard"></td>
</tr>
</tbody>
</table>

### Undo and Revert Undo Operations

Starting with release 2018.46.x, you can undo dashboard changes.

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<ol><li>Edit the dashboard and make one or more changes.</li>
<li>Click the icons to undo and to revert the undo. </li> </ol></td>
<td width="60%"><img src="/images/v2_undo.png" alt="Undo and redo icons"/></td>
</tr>
</tbody>
</table>

### Set Dashboard Display Preferences

For each dashboard, you can customize display preferences.

{% include tip.html content="To use the dark theme with your dashboard and all other Wavefront UI components, set your personal preferences [from the gear icon](users_account_managing.html#configuring-user-preferences)."%}

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<strong>To set the dashboard display preferences</strong>:
<ol><li>Navigate to a dashboard, click the ellipsis menu in the top right corner of the dashboard.</li>
<li>Click <strong>Edit</strong>. </li>
<li>Click <strong>Settings</strong>.</li>
<li>Make selections in the dialog:
<ul><li>Set the default time window. You can later override the time window.  </li>
<li>Hide the variables for the dashboard by default. Users can still show the variables bar using the <img src="/images/show_hide_variable_icon.png"
style="vertical-align:text-bottom;width:25px" alt="show hide variable icon" /> icon.  </li>
<li>Display the <a href="events.html">Events</a> on charts using the Show Events dropdown.<br>
For more information on the options listed in the Show Events dropdown, see <a href="charts_events_displaying.html#specify-an-events-query-for-a-dashboard">Specify an Events() Query for a Dashboard</a>
</li>
</ul>
</li>
<li>Click <strong>Accept</strong>, and click <strong>Save</strong> </li>
</ol></td>
<td width="60%"><img src="/images/v2_dashboard_prefs.png" alt="set dashboard prefs"></td>
</tr>
</tbody>
</table>



{% include shared/system_dashboard.html %}

## Advanced: Edit Dashboard JSON

Most users create and edit dashboards by using the Wavefront UI or automate the process with the Wavefront REST API. But at times, it's convenient to edit the dashboard JSON directly from the UI and see results immediately. This section shows how to make changes from the dashboard JSON editor. We use a simple example of adding a conditional section to illustrate how it works.


### Example: Create Conditional Dashboard Sections

Imagine you have a dashboard with many sections. Several of them are relevant only if certain metrics are currently shown in the charts of that dashboard section. You can conditionalize the dashboard to show a specified section only under certain conditions.

{% include warning.html content="Editing the dashboard JSON might have unintended consequences. Use the JSON editor only if you have some experience with JSON. " %}

<table style="width: 100%;">
<tbody>
<tr>
<td width="40%">
<ol><li>To put the dashboard in Edit mode, click the three dots and select <strong>Edit</strong>, and then click <strong>JSON</strong>. </li>
<li>Consider selecting Code view from the pull-down menu. Code view supports adding information. </li>
<li>Consider a select all/copy/paste into a JSON editor for full validation. </li>
<li>Add condition information, as shown in the example below, paste the revised content back into the dashboard editor, and click <strong>Accept</strong></li></ol></td>
<td width="60%"><img src="images/dashboard_code_view.png" alt="Switch from Tree view to Code view"/></td>
</tr>
</tbody>
</table>


The following sample JSON includes a `sectionFilter` property that lets you show or hide the section based on a condition.

```
sections: [
  {
    name: string  // Section name, if not specified, the text "Untitled" is shown in the section header
    rows: array   // Array of visual components in this section
    // This property is optional.  If specified, then query is required.
    sectionFilter: {
      // Query to run to determine if the section should be shown.  The section is shown if the last
      // value in the time series is non-zero.
      query: string
      // Text to show as tooltip when users mouse over the condition check-circle icon.
      // If not specified, then the query is shown as the tooltip.
      // This property is optional.
      description: string
      // Time in seconds to add to start time for condition query.  By default, condition query uses
      // the dashboard time window, but you can use this property to increase the time window of the
      // condition query.
      // This property is optional.
      leadingTimeWindowSec: integer
    }
  }
]
```


The following JSON snippet allows you to Modify the Jump To label in case a condition specified by `sectionFilter` has been met. Badge colors correspond to colors used by alerts.

```
dashboardAttributes: {
  // Text to replace the "Jump To" label on the dashboard view page.
  // This property is optional.
  jumpToLabel: string
  // When section conditions are met, a count badge is rendered to the right of the "Jump To"
  // dropdown control.  The default badge style is SMOKE.  Users can customize this style by setting
  // the badge color here.  Valid values are SEVERE, WARN, INFO, SUCCESS, and SMOKE.
  // This property is optional.
  conditionBadgeColor: string
}
```
