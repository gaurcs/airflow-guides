---
title: "Google Analytics API to Redshift with Airflow"
description: "Use Airflow to ingest data from the Google Analytics API to Redshift"
date: 2018-05-21T00:00:00.000Z
slug: "google-analytics-to-redshift"
heroImagePath: "https://assets.astronomer.io/website/img/guides/GoogleAnalyticsToRedshift_preview.png"
tags: ["DAGs", "Integrations"]
---

In this guide, we’ll explore how you can use [Apache Airflow](https://airflow.apache.org/) to move your data from Google Analytics to Redshift. Note that this is an effective and flexible alternative to point-and-click ETL tools like [Segment](https://segment.com), [Alooma](https://alooma.com), [Xplenty](https://xplenty.com), [Stitch](https://stitchdata.com), and [ETLeap](https://etleap.com/).

Before we get started, be sure you have the following on hand:

* A Google Analytics account
* An S3 bucket with a valid `aws_access_key_id` and `aws_secret_access_key`
* A Redshift instance with a valid host IP and login information
* An instance of Apache Airflow. You can either set this up yourself if you have devops resources or sign
  up and get going immediately with Astronomer’s managed Airflow service. Note that this guide will use
  commands using the Astronomer CLI to push dags into production and will assume you’ve spun up an Airflow
  instance via Astronomer, but the core code should work the same regardless of how you’re hosting Airflow
* Docker running on your machine

> **Note:** In Airflow 2.0, provider packages are separate from the core of Airflow. If you are running 2.0 with Astronomer, the `apache-airflow-providers-amazon` package is already included in our Astronomer Certified Image. If you are not using Astronomer, you may need to install the package separately to use the hooks, operators, and connections described here. To learn more, read [Airflow Docs on Provider Packages](https://airflow.apache.org/docs/apache-airflow-providers/index.html).

This DAG generates a report using v4 of the Google Analytics Core Reporting API. The dimensions and metrics are as follows. Note that while these can be modified, a maximum of 10 metrics and 7 dimensions can be requested at once.

METRICS

* pageView
* bounces
* users
* newUsers
* goal1starts
* goal1completions

DIMENSIONS

* dateHourMinute
* keyword
* referralPath
* campaign
* sourceMedium

Not all metrics and dimensions are compatible with each other. When forming the request, please refer to the official [Google Analytics API Reference docs](https://developers.google.com/analytics/devguides/reporting/core/dimsmets)

### 1. Add Connections in Airflow UI

Begin by creating all of the necessary connections in your Airflow UI. To do this, log into your Airflow dashboard and navigate to Admin-->Connections. In order to build this pipeline, you’ll need to create a connection to your Google Analytics account, your S3 bucket, and your Redshift instance. For more info on how to fill out the fields within your connections, check out our [documentation here](https://www.astronomer.io/guides/connections/).

### 2. Clone the plugin

If you haven't done so already, navigate into your project directory and create a `plugins` folder by running  `mkdir plugins` in your terminal.Navigate into this folder by running `cd plugins` and clone the [Google Analytics Plugin](https://github.com/airflow-plugins/google_analytics_plugin) using the following command:

`git clone https://github.com/airflow-plugins/google_analytics_plugin.git`

This will allow you to use the Google Analytics hook to establish a connection to Google Analytics and extract data into a file. You will also be able to use the appropriate operators to transfer the Google Analytics data to S3 and then from S3 to Redshift.

### 3. Copy the DAG file

Navigate back into your project directory and create a `dags` folder by running `mkdir dags`. Copy the [Google Analytics to Redshift DAG file](https://github.com/airflow-plugins/Example-Airflow-DAGs/blob/master/etl/google_analytics_to_redshift.py) into this folder.

### 4. Customize

Open up the [google_analytics_to_redshift.py file](https://github.com/airflow-plugins/Example-Airflow-DAGs/blob/master/etl/google_analytics_to_redshift.py#L46) that you just copied in a text editor of your choice and input the following credentials into lines 46-50:

```python
S3_CONN_ID = ''
S3_BUCKET = ''
GOOGLE_ANALYTICS_CONN_ID = ''
REDSHIFT_CONN_ID = ''
REDSHIFT_SCHEMA = ''
```

### 5. Test + Deploy

Once you have those credentials plugged into your DAG, test and deploy it!

If you don't have Airflow already set up in your production environment, head over to [our getting started guide](https://astronomer.io/docs/getting-started) to get spun up with your own managed instance!
