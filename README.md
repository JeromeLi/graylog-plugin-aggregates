# Aggregates Plugin for Graylog

[![Build Status](https://travis-ci.org/cvtienhoven/graylog-plugin-aggregates.svg?branch=master)](https://travis-ci.org/cvtienhoven/graylog-plugin-aggregates)

**Required Graylog version:** 2.3.0 and later - **NOT BACKWARDS COMPATIBLE WITH OLDER GRAYLOG VERSIONS**.
For the latest version compatible with Graylog 2.2.x, use version **[1.0.1](https://github.com/cvtienhoven/graylog-plugin-aggregates/releases/tag/1.0.1)** of the plugin.


**_Note_**: When upgrading from 1.x.x to version 2.0.0 of the plugin, it's required to modify existing rules (regarding streams and alerting) and existing report schedules (regarding receivers). See the screenshots below.



The Aggregates Plugin for Graylog enables users to execute term searches and get notified when the given criteria are met. Currently, there are the following alert conditions in Graylog:

* Message count
* Field value
* Field content

However, these conditions will not be sufficient to match the following scenario:

**_Send an alert when someone fails to login from the same source ip 3 or more times in the past minute._**

This scenario is actually very useful in a security context, but with the built-in alert conditions, it's not possible to match exactly this condition. The part "**_from the same source ip_**" is difficult to match. It would take an aggregate search that groups by value and returns the count per value. That's what the plugin aims to do. You can configure rules as shown in the screenshots below.

**Create / edit a rule**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/edit_rule.png)


**Generated Alert Condition**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/condition.png)


**Alert Overview**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/alert.png)


**Rule overview**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/list.png)


**Report schedule overview**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/schedule_list.png)


**Create / edit a report schedule**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/edit_schedule.png)


**Report example**

![](https://github.com/cvtienhoven/graylog-plugin-aggregates/blob/master/images/report.png)


Usage
-----

**Permissions**

Use the Aggregates tab in the web interface of Graylog to define rules with alert criteria. For non-admin users, there are the following permissions that should be configured (via the REST API) to be able to fully (or partly) operate the plugin:

* aggregate_rules:create
* aggregate_rules:read
* aggregate_rules:update
* aggregate_rules:delete

* aggregate_report_schedules:create
* aggregate_report_schedules:read
* aggregate_report_schedules:update
* aggregate_report_schedules:delete

**_Note_**: To be able to view the list of rules, non-admin users need both the `aggregate_rules:read` and the `aggregate_report_schedules:read` permissions.

Each rule can be configured to be executed on a particular stream. For the latter option to be present, the user needs to be able to have at least the following permissions:

* searches:absolute
* searches:relative
* searches:keyword

**Sending alerts**

Since version 2.0.0, the plugin integrates tightly with the `Notifications` and `Alert Conditions` within Graylog. You can define a notification on a stream as you would normally do. The plugin creates an Alert Condition when creating an Aggregate Rule and it keeps the condition in sync with the rule after updates. In previous versions, you could only send emails, but now you can also use the HTTP Alarm Callback for instance. If you still want to use emails, you'll have to use the Email Alarm Callback. Unfortunately the HTML markup in emails had to be discarded since the Email Alarm Callback sends emails in plain text.

**_Note 1_**: If you delete the Alert Condition, the plugin re-creates it, except when you disable the rule.<br/>
**_Note 2_**: Enabling the message backlog can inflict a performance penalty, as the backlog has to be assembled from the found terms, using separate searches. Use with care.<br/>
**_Note 3_**: Alert Conditions are created under the user `admin`.


**Reporting**

In the rule overview, there's an option (checkbox) to include rule history in a report. This report is a PDF file that contains a bar chart for every rule, summing up the total number of hits for that rule per period. The grid for the chart is automatically determined based on the total amount of time. The report is tailored per alert receiver, which means that a receipient will only receive charts for the rules subscribed to.

When creating or editing a rule, the schedule(s) for generating report(s) can be supplied. For configuring a schedule, you should supply a name, a valid Cron expression using the [Drools](http://javadox.com/org.drools/drools-core/6.2.0.Final/org/drools/core/time/impl/CronExpression.html) syntax and the timespan, e.g. the amount of history you wish to incorporate in the report. Since version 2.0.0 of the plugin, receivers of reports are defined on the report schedule, not on the aggregates rule anymore.

**_Note_**: The maximum timespan determines the overall retention. So, if you have a report schedule that takes a year of history, the retention of hits will be a year. This might influence your MongoDB storage needs.


# Changelog

2.1.1
-----

- Bugfix: Creation of new rules failed with error 500
- Bugfix: With only one available stream, corrupt rules got created
- Bugfix: In the API browser, duplicate entries existed for Plugin/Aggregates (#21)

2.1.0
-----

- Fully integrated alerting with the native Graylog notifications method, including message backlog (#24, #25)
- Removed the "sliding" parameter, because the generated AlertCondition is always evaluated every minute
- Fixed bug that renaming a rule wasn't reflected in history
- Implemented functionality so no results would trigger an alert if theres a "less than" condition (#11)

2.0.0
-----

- Moved report subscriber option to the Report Schedules screen
- Introduced alerting via the native Graylog Alarm Callbacks (Notifications)




Getting started
---------------

This project is using Maven 3 and requires Java 8 or higher.

* Clone this repository.
* Run `mvn package` to build a JAR file.
* Optional: Run `mvn jdeb:jdeb` and `mvn rpm:rpm` to create a DEB and RPM package respectively.
* Copy generated JAR file in target directory to your Graylog plugin directory.
* Restart the Graylog.

Plugin Release
--------------

We are using the maven release plugin:

```
$ mvn release:prepare
[...]
$ mvn release:perform
```

This sets the version numbers, creates a tag and pushes to GitHub. Travis CI will build the release artifacts and upload to GitHub automatically.

Installation
------------

[Download the plugin](https://github.com/cvtienhoven/graylog-plugin-aggregates/releases)
and place the `.jar` file in your Graylog plugin directory. The plugin directory
is the `plugins/` folder relative from your `graylog-server` directory by default
and can be configured in your `graylog.conf` file.

Restart `graylog-server` and you are done.

Development
-----------

You can improve your development experience for the web interface part of your plugin
dramatically by making use of hot reloading. To do this, do the following:

* `git clone https://github.com/Graylog2/graylog2-server.git`
* `cd graylog2-server/graylog2-web-interface`
* `ln -s $YOURPLUGIN plugin/`
* `npm install && npm start`
