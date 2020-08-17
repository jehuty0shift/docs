---
title: Push and vizualize your first datapoints with OpenTSDB
slug: start-opentsdb
excerpt: Push and query your first datapoints
section: Getting started
order: 3
---

**Last updated 23th August, 2019**

## Objective

[OpenTSDB](http://opentsdb.net/){.external} is a Scalable Time Series Database design to store and serve massive amounts of time series data without losing granularity. In this guide, you will learn how to use your first datapoints using Metrics.

## Requirements

- a valid OVH Metrics account.
- a `READ` token that can be found on Metrics`s manager
- a `WRITE` token that can be found on Metrics`s manager
- curl installed on your machine

## Instructions

### Create a JSON to push data

The full documentation is available at [http://opentsdb.net/docs/build/html/api_http/put.html](http://opentsdb.net/docs/build/html/api_http/put.html){.external}. 
As an example you can push single point.

OpenTSDB is accepting a JSON containing the datapoints. Here's an example:

```json
  {
    "metric": "sys.cpu.nice",
    "timestamp": 1346846400,
    "value": 18,
    "tags": {
        "host": "web01",
        "dc": "lga"
    }
  }
```

You can also push multiple datapoints into a single query. Create a file called `opentsdb.json` and write this content to it:

```json
[
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346846400000,
        "value": 18,
        "tags": {
           "host": "web01",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346846800000,
        "value": 19,
        "tags": {
           "host": "web01",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346847000000,
        "value": 20,
        "tags": {
           "host": "web01",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346847000005,
        "value": 19,
        "tags": {
           "host": "web01",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346846400000,
        "value": 9,
        "tags": {
           "host": "web02",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346846800000,
        "value": 8,
        "tags": {
           "host": "web02",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346847000000,
        "value": 7,
        "tags": {
           "host": "web02",
           "dc": "lga"
        }
    },
    {
        "metric": "sys.cpu.nice",
        "timestamp": 1346847000005,
        "value": 10,
        "tags": {
           "host": "web02",
           "dc": "lga"
        }
    }
]
```

#### Bash &amp; curl

In the `curl` command below, replace __REGION__ per your own one : __gra1__  or __bhs1__. As the user doesn't matter on the metrics backend, all the information are stored in our cryptographical token, you can replace or let DESC as if.

```shell-session
$ curl -X POST --data-binary @opentsdb.json 'https://DESC:TOKEN_WRITE@opentsdb.REGION.metrics.ovh.net/api/put'
```

If everyting happens correctly, the CURL would exit with a 200 code status. You can check that you correctly upload the data by checking your OVH metrics dashboard. In your home panel, you should now have 2 more active series and 6 new points added.


### Query data with OpenTSDB

Now let's retrieve the previously pushed data.

The full documentation is available at [http://opentsdb.net/docs/build/html/api_http/query/index.html](http://opentsdb.net/docs/build/html/api_http/query/index.html){.external}

Let's write a `query.json` file which contains the following code:

```json
{
    "start": 1346846000000,
    "end": 1346847300005,
    "queries": [{
        "metric": "sys.cpu.nice",
        "aggregator": "min",
        "downsample": "4m-avg",
        "tags": {
            "host": "*",
            "dc": "*"
        }
    }]
}
```

This will get all the saved points and compute the query before returning the result. The curl command to execute this query is:

```shell-session
$ curl --data-binary @query.json 'https://DESC:TOKEN_READ@opentsdb.REGION.metrics.ovh.net/api/query'
```

You should expects a result similar to:

```json
[{"metric":"sys.cpu.nice","tags":{"dce":"lga", "host": "web02"},"aggregateTags":["host","de"],
"dps": {"1346846340" :9, "1346846580" :9, "1346846820" :8, "1346847060" :8.5, "1346847300" :8.5}},{"m
etric":"sys.cpu.nice","tags":{"dce":"lga","host":"web01"},"aggregateTags":["host","dc"], "dps
":{"13468463460" :18, "1346846580" :18, "1346846820" :19, "1346847060" :19.5, "1346847300" :19.5}}]
```

What does compute this query on the series? 

The series are split according to the downsample parameter (4m-avg) starting from the end time. Then in each 4 minutes periods, compute the average on all values and return one result per period. This function performs a linear interpolation on the series and will fill each missing periods between the start and end time of the series.

With this request the aggregator parameter wasn't used. To compute an aggregation over several metrics you can specify the following request where the host is removed:

```json
{
    "start": 1346846000000,
    "end": 1346847300005,
    "queries": [{
        "metric": "sys.cpu.nice",
        "aggregator": "min",
        "downsample": "4m-avg",
        "tags": {
            "dc": "*"
        }
    }]
}
```

Then the following curl command:

```shell-session
$ curl --data-binary @query.json 'https://DESC:TOKEN_READ@opentsdb.REGION.metrics.ovh.net/api/query'
```

provides this result:

```json
[{"metric":"sys.cpu.nice","tags":{"dc":"lga"},"aggregateTags":["dc"],"dps":{"1346846340":9,
"1346846580" :9, "1346846820" :8, '"1346847060":8.5, "1346847300" :8.5}}]
```

This provide only a series as result containing the minimal value for each 4 minutes periods. You will find more details about OpenTSDB downsampling and aggregation [here](http://opentsdb.net/docs/build/html/user_guide/query/aggregators.html){.external}.

### Datasource setup

OpenTSDB is integrated with Grafana : [Read more on OpenTSDB builtin data source](http://docs.grafana.org/features/datasources/opentsdb/){.external}.

The first thing you need is to create a new OpenTSDB datasource. You can deploy your own grafana or create a new dashboard on the [Public OVH Grafana instance](https://grafana.metrics.ovh.net){.external}.

Go to :

__Menu__ > __Data sources__ > __Add datasource__

![menu](images/grafana-menu-datasource.png){.thumbnail}

How to fill the blanks:

* `Name` field is free.
* `Type` is __OpenTSDB__.
* `URL` depends of your project region it's build as this https://opentsdb.REGION.metrics.ovh.net where region can be __gra1__ or __bhs1__. You can refer to the OVH manager top find your region.
* `Access` is __direct__.
* `Basic Auth` checkbox must be checked.
* `user` is not used, you can use __metrics__ as basic auth.
* `password` must be fileld with a Metrics Read token. Tokens can be crafted on the manager.
* `version` can be set to __<=2.1__

![menu](images/grafana-datasource-opentsdb.png){.thumbnail}

### Simple query

You can add a new graph panel by clicking on:

__Menu__ > __Dashboard__ > __New__

Then add a new graph panel, and choose your data source by his name.

![menu](images/grafana-opentsdb-query.png){.thumbnail}

You can put your metrics into the `Metric name` field.

If you see too much variations and precision on your graph, decrease the `Down sample` interval.
if your graaph is too blur, increase the `Down sample` interval.

Let's now play we the previous example we pushed in metrics. First of all the timestamp we put corresponds to the 05 september 2012 around 11h UTC (so 13 CEST). Select a custom time on the top right of Graphana. Chose as example a time window between 11h CEST and 15h CEST. You should have something similar to:

![menu](images/opentsdb-grafana-time.png){.thumbnail}

Then we add a query on the graph to query both metrics. You just have to put `sys.cpu.nice` as metric name, then fill each parameters to select the aggregator, the downsample and the tag. In our case we can do a single queries to get both cpu series just by adding the following tag `host=*`.

![menu](images/opentsdb-grafana-result.png){.thumbnail}

### Delete data

To delete all your example data you can use the simple Warp 10™ delete command used below

```shell-session
$ curl -H 'X-Warp10-Token: WRITE_TOKEN' 'https://warp10.gra1.metrics.ovh.net/api/v0/delete?deleteall&selector=sys.cpu.nice\{\}'
```

Once you delete data in metrics, you will not be able to retrieve them. Be careful with this command. It will also not reduced your current count of daily points pushed.

## Go further

- To learn more about pushing and querying with OpenTSDB, you can follow our protocol guide [use OpenTSDB](../protocol_opentsdb/){.ref}.
- Documentation: [Guides](../product.fr-fr.md){.ref}
- Vizualize your data: [https://grafana.metrics.ovh.net/login](https://grafana.metrics.ovh.net/login){.external}
- Community hub: [https://community.ovh.com](https://community.ovh.com/c/platform/data-platforms){.external}
- Create an account: [Try it free!](https://www.ovh.com/fr/order/express/#/new/express/resume?products=~%28~%28planCode~%27metrics-free-trial~configuration~%28~%28label~%27region~values~%28~%27gra1%29%29%29~option~%28~%29~quantity~1~productId~%27metrics%29%29&paymentMeanRequired=0){.external}
