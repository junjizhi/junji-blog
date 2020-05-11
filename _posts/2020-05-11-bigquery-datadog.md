---
layout: post
title: "Send BigQuery Query Results to Datadog?"
categories: [All]
tags: [BigQuery, Datadog, Google Data Studio]
fullview: false
excerpt: How I solved this problem with Google Data Studio
comments: true
---

## The Problem

We found disprepencies between two datasets and go on a journey to fix those disprepencies. Also, we would like visualize disprepencies. 

Since we already have custom SQL to BigQuery, and we are already using Datadog, the idea is run the query every once a while and send the query results
as custom metrics to Datadog. 

I was looking for out of box solution, but it turns out there isn't any. 

Current BigQuery + Datadog [integration documentation](https://docs.datadoghq.com/integrations/google_cloud_big_query/) is all about collecting system metrics. Datadog
 has the support to run custom queries against [MySQL](https://docs.datadoghq.com/integrations/faq/how-to-collect-metrics-from-custom-mysql-queries/), but there isn't anything like
 that for BigQuery.

## Temporary Solution

 We ended up using [Google Data Studio](https://datastudio.google.com/reporting). Their [documentation](https://cloud.google.com/bigquery/docs/visualize-data-studio) is a bit 
 concise, and it took me a little time to configure the a simple report to pull data from BigQuery.

Manage data source:

 ![image](https://user-images.githubusercontent.com/2715151/81616129-f3152d80-93b0-11ea-80ed-5a9eb447a6eb.png)




Enter custom query (Note the CUSTOM QUERY tab on the left):

 ![image](https://user-images.githubusercontent.com/2715151/81616208-163fdd00-93b1-11ea-9d71-e3a4d1c83ff0.png)


It took a little time to adjust to their UI, but essentially it is like building SQL queries:

![image](https://user-images.githubusercontent.com/2715151/81616587-c0b80000-93b1-11ea-8e6e-2e629f86f683.png)


## Closing Thoughts
With Google Data Studio, we can at least visualize the count of records that we are interested in. Also, it
allows us to [set the refreshness](https://support.google.com/datastudio/answer/7020039?hl=en) for the data source, or
manually refresh it. So it satisfies our need in the short term. 

However, we can't set up any alert from there like we do in Datadog. It would be nice to see more integration
support from Datadog.