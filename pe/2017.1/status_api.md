---
layout: default
title: "Monitoring PE services"
canonical: "/pe/latest/status_api.html"
---

[tech preview]: https://puppetlabs.com/services/tech-preview

## Puppet Services status monitor

The Puppet Services status monitor provides a visual overview of the current state of the following key Puppet services:

* Activity service
* Node classifier
* PuppetDB
* Puppet Server
* Role-based access control (RBAC)
* Code Manager (if configured)

To view the Puppet Services status monitor, navigate to the **Overview** page in the console. In the upper right corner, a checkmark appears next to **Puppet Services status** if all applicable services are accepting requests. In the event that no data is available, a question mark appears next to the link. If one or more services is restarting or not accepting requests, a warning icon appears.

To view the status of each service, click **Puppet Services status** to open the monitor.

## `puppet infrastructure status` command

The `puppet infrastructure status` command displays errors and alerts from PE services, including the activity, classifier, and RBAC services, Puppet Server, and PuppetDB. If [high availability](./ha_overview.html) is enabled in your environment, the command reports separately on the primary master, primary master replica, and compile masters.


## The status API

The status endpoints allow you to check the health of PE components. You can check the overall health of pe-console-services, as well as the health of the individual services within pe-console-services:

* [Activity service](./rbac_activityapis.html)
* [Node classifier](./nc_index.html)
* [PuppetDB]({{puppetdb}}/api/index.html)
* [Puppet Server](./api_index.html#puppet-server-administrative-api)
* [Role-based access control (RBAC)](./rbac_serviceindex_v1.html)
* [Code Manager](./api_index.html#code-manager-api) (if configured)

The endpoints provide both overview health information in an overall healthy/error/unknown status field, and fine-detail information such as the availability of the database, the health of other required services, or the connectivity to the Puppet master. It can be useful in automated monitoring of your PE infrastructure, removing unhealthy service instances from a load-balanced pool, checking configuration values, or when troubleshooting problems in PE.

### Authenticating requests

You need to authenticate requests to the status API using a certificate listed in RBAC's certificate whitelist, located at `/etc/puppetlabs/console-services/rbac-certificate-whitelist`. Note that if you edit this file, you must reload the pe-console-services service for your changes to take effect (`sudo service pe-console-services reload`). You can attach the certificate using the command line as demonstrated in the example cURL query in [Forming requests](#forming-requests). You must have the whitelist certificate name and the private key to run the script.

The status endpoints can be served over HTTP, which does not require any authentication. This is disabled by default. To enable it from the PE console, go to the **PE Console** node group, and in the **puppet_enterprise::profile::console** class, set the **console_services_plaintext_status_enabled** parameter to **true**.

### Forming requests

The HTTPS status endpoints are available on the console services API server, which uses port 4433 by default.

The path prefix is `/status`, so, for example, the URL to get the statuses for all services as JSON is `https://<DNS NAME OF YOUR CONSOLE HOST>:4433/status/v1/services`.

To access that URL using cURL, run:

~~~
curl https://<DNS NAME OF YOUR CONSOLE HOST>:4433/status/v1/services \
    --cert /etc/puppetlabs/puppet/ssl/certs/<WHITELISTED CERTNAME>.pem \
    --key /etc/puppetlabs/puppet/ssl/private_keys/<WHITELISTED CERTNAME>.pem \
    --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem
~~~

If enabled, the HTTP status endpoints are available on port 8123. To change this port, in the PE console UI, go to the `PE Console` node group, and in the `puppet_enterprise::profile::console` class, set the `console_services_plaintext_status_port` parameter to your desired port number.

The path prefix is `/status`, so for example, the URL to get the statuses for all services as JSON is `http://<DNS NAME OF YOUR CONSOLE HOST>:8123/status/v1/services`.

To access that URL using cURL, run:

    curl http://<DNS NAME OF YOUR CONSOLE HOST>:8123/status/v1/services

### JSON endpoints

These two endpoints provide machine-consumable information about running services. They are intended for scripting and integration with other services.

The content type for these endpoints is `application/json; charset=utf-8`.

**Response keys**

* **service_version:** Package version of the JAR file containing a given service.
* **service_status_version:** Version of the status API the service is using to report status.
* **state:** One of _running_, _error_, or _unknown_
* **detail_level:** How thorough of a check to be run. One of _critical_, _debug_, _info_.
* **status:** An object with service’s status details. Usually only relevant for _error_ and _unknown_ statuses.
* **active_alerts**: An array of objects containing `severity` and a `message` about your replication from pglogical if you have replication enabled; otherwise, it's an empty array.

#### GET /status/v1/services

**Query parameters**

- `level` (Optional) The possible values are: `critical`, `debug`, `info`. Defaults to `info`.
- `timeout` (Optional) The timeout period, expressed in seconds. Defaults to 30. 

Returns statuses for all services the status service knows about in a JSON data structure.

**Response codes**

* 200 if and only if all services report a status of _running_
* 503 if any service's status is _unknown_ or _error_
* 400 if a level parameter is set but is invalid (not _critical_, _debug_, or _info_)

**Example response**

~~~ json
{"rbac-service": {"service_version": "1.8.11-SNAPSHOT",
                  "service_status_version": 1,
                  "detail_level": "info",
                  "state": "running",
                  "status": {
                    "activity_up": true,
                    "db_up": true,
                    "db_pool": { "state": "ready" },
                    "replication": { "mode": "none", "status": "none" }
                  },
                  "active_alerts": [],
                  "service_name": "rbac-service"
                  }


 "classifier-service": {"service_version": "1.8.11-SNAPSHOT",
                  "service_status_version": 1,
                  "detail_level": "info",
                  "state": "running",
                  "status": {
                    "activity_up": true,
                    "db_up": true,
                    "db_pool": { "state": "ready" },
                    "replication": { "mode": "none", "status": "none" }
                  },
                  "active_alerts": [],
                  "service_name": "classifier-service"
                  }
~~~
#### GET /status/v1/services/&lt;SERVICE NAME&gt;

**Query parameters**

- `level` The possible values are: `critical`, `debug`, `info`. Defaults to `info`.
- `timeout` (Optional) The timeout period, expressed in seconds. Defaults to 30. 

Returns the status of the specified service, such as "rbac-service" or "classifier-service".

**Response codes**

* 200 if the service reports a status of _running_
* 503 if the service reports a status of _unknown_ or _error_
* 404 if no service named &lt;SERVICE NAME&gt; is found
* 400 if a level parameter is set but is invalid (not _critical_, _debug_, or _info_)

**Example response**

~~~json
{"rbac-service": {"service_version": "1.8.11-SNAPSHOT",
                  "service_status_version": 1,
                  "detail_level": "info",
                  "state": "running",
                  "status": {
                    "activity_up": true,
                    "db_up": true,
                    "db_pool": { "state": "ready" },
                    "replication": { "mode": "none", "status": "none" }
                  },
                  "active_alerts": [],
                  }
~~~

### Plaintext endpoints

These two endpoints are designed for load balancers that don't support any kind of
JSON parsing or parameter setting. They return simple string bodies (either the
state of the service in question or a simple error message) and a status code
relevant to the status result.

The content type for these endpoints is `text/plain; charset=utf-8`.

#### GET /status/v1/simple

Returns a status that reflects all services the status service knows about. It
decides on what status to report using the following logic:

* _running_ if and only if all services are _running_.
* _error_ if any service reports _error_.
* _unknown_ if any service reports _unknown_ and no services report _error_.

**Query parameters**

No parameters are supported. Defaults to using the _critical_ status level.

**Response codes**

* 200 if and only if all services report a status of _running_
* 503 if any service's status is _unknown_ or _error_

**Possible responses**

* "running"
* "error"
* "unknown"

#### GET /status/v1/simple/&lt;SERVICE NAME&gt;

Returns the plaintext status of the specified service, such as "rbac-service" or "classifier-service".

**Query parameters**

No parameters are supported. Defaults to using the _critical_ status level.

**Response codes**

* 200 if service is _running_
* 503 if service is _unknown_ or _error_
* 404 if requested service is not found

**Possible responses**

* "running"
* "error"
* "unknown"
* "not found: &lt;SERVICE NAME&gt;"

### Metrics endpoints

Puppet Server is capable of tracking advanced metrics to give you additional insight into its performance and health.

The HTTPS metrics endpoints are available on port 8140 of the master server:

    curl -k https://<DNS NAME OF YOUR MASTER>:8140/status/v1/services?level=debug`

> **Note:** These API endpoints are a [tech preview][]. The metrics described here are returned only when passing the `level=debug` URL parameter, and the structure of the returned data might change in future versions.

These metrics fall into three categories:

* JRuby metrics
* HTTP route metrics
* Catalog compilation profiler metrics

All of these metrics reflect data for the lifetime of the current Puppet Server process and reset whenever the service is restarted. Any time-related metrics report milliseconds unless otherwise noted.

Like the standard Status API endpoints, the metrics endpoints return machine-consumable information about running services. This JSON response includes the same keys returned by a standard status endpoint request (see [JSON endpoints](#json-endpoints)). Each endpoint also returns additional keys in an `experimental` section.

#### GET /status/v1/services/pe-jruby-metrics

Returns JSON containing information about the JRuby pools from which Puppet Server fulfills agent requests. As with all other metrics endpoints, you must query it at port 8140 and append the `level=debug` URL parameter.

**Query parameters**

No parameters are supported. Defaults to using the _critical_ status level.

**Response codes**

* 200 if and only if all services report a status of _running_
* 503 if any service's status is _unknown_ or _error_

**Response keys**

The metrics are returned in two subsections of the `experimental` section: `jruby-pool-lock-status` and `metrics`.

The response's `experimental/jruby-pool-lock-status` section contains the following keys:

* **current-state:** The state of the JRuby pool lock, which should be either _":not-in-use"_ (unlocked), _":requested"_ (waiting for lock), or _":acquired"_ (locked).
* **last-change-time:** The date and time of the last `current-state` update, formatted as an ISO 8601 combined date and time in UTC.

The response's `experimental/metrics` section contains the following keys:

* **average-borrow-time:** The average amount of time a JRuby instance spends handling requests, calculated by dividing the total duration in milliseconds of the `borrowed-instances` value by the `borrow-count` value.
* **average-free-jrubies:** The average number of JRuby instances that are not in use over the Puppet Server process's lifetime.
* **average-lock-held-time:** The average time the JRuby pool held a lock, starting when the value of `jruby-pool-lock-status/current-state` changed to _":acquired"_. This time mostly represents [file sync](./cmgmt_filesync.html) syncing code into the live codedir, and is calculated by dividing the total length of time that Puppet Server held the lock by the value of `num-pool-locks`.
* **average-lock-wait-time:** The average time Puppet Server spent waiting to lock the JRuby pool, starting when the value of `jruby-pool-lock-status/current-state` changed to _":requested"_). This time mostly represents how long Puppet Server takes to fulfill agent requests, and is calculated by dividing the total length of time that Puppet Server waits for locks by the value of `num-pool-locks`.
* **average-requested-jrubies:** The average number of requests waiting on an available JRuby instance over the Puppet Server process's lifetime.
* **average-wait-time:** The average time Puppet Server spends waiting to reserve an instance from the JRuby pool, calculated by dividing the total duration in milliseconds of `requested-instances` by the `requested-count` value.
* **borrow-count:** The total number of JRuby instances that have been used.
* **borrow-retry-count:** The total number of times that a borrow attempt failed and was retried, such as when the JRuby pool is flushed while a borrow attempt is pending.
* **borrow-timeout-count:** The number of requests that were not served because they timed out while waiting for a JRuby instance.
* **borrowed-instances:** A list of the JRuby instances currently in use, each reporting:
	* **duration-millis:** The length of time that the instance has been running.
	* **reason/request:** A hash of details about the request being served.
		* *request-method:* The HTTP request method, such as post, get, put, or delete.
		* *route-id:* The route being served. For routing metrics, see [the HTTP metrics endpoint](#get-statusv1servicespe-master).
		* *uri:* The request's full URI.
	* **time:** The time (in milliseconds since the [Unix epoch](https://en.wikipedia.org/wiki/Unix_time)) when the JRuby instance was borrowed.
* **num-free-jrubies:** The number of JRuby instances in the pool that are ready to be used.
* **num-jrubies:** The total number of JRuby instances.
* **num-pool-locks**: The total number of times the JRuby pools have been locked.
* **requested-count:** The number of JRuby instances borrowed, waiting, or that have timed out.
* **requested-instances:** A list of the requests waiting to be served, each reporting:
	* **duration-millis:** The length of time the request has waited.
	* **reason/request:** A hash of details about the waiting request.
		* *request-method*: The HTTP request method, such as post, get, put, or delete.
		* *route-id*: The route being served. For routing metrics, see [the HTTP metrics endpoint](#get-statusv1servicespe-master).
		* *uri*: The request's full URI.
	* **time:** The time (in milliseconds since the [Unix epoch](https://en.wikipedia.org/wiki/Unix_time)) when Puppet Server received the request.
* **return-count:** The total number of JRuby instances that have been used.

**Example response**

~~~ json
"pe-jruby-metrics": {
    "detail_level": "debug",
    "service_status_version": 1,
    "service_version": "2.2.22",
    "state": "running",
    "status": {
        "experimental": {
            "jruby-pool-lock-status": {
                "current-state": ":not-in-use",
                "last-change-time": "2015-12-03T18:59:12.157Z"
            },
            "metrics": {
                "average-borrow-time": 292,
                "average-free-jrubies": 0.4716243097301104,
                "average-lock-held-time": 1451,
                "average-lock-wait-time": 0,
                "average-requested-jrubies": 0.21324752542875958,
                "average-wait-time": 156,
                "borrow-count": 639,
                "borrow-retry-count": 0,
                "borrow-timeout-count": 0,
                "borrowed-instances": [
                    {
                        "duration-millis": 3972,
                        "reason": {
                            "request": {
                                "request-method": "post",
                                "route-id": "puppet-v3-catalog-/*/",
                                "uri": "/puppet/v3/catalog/hostname.example.com"
                            }
                        },
                        "time": 1448478371406
                    }
                ],
                "num-free-jrubies": 0,
                "num-jrubies": 1,
                "num-pool-locks": 2849,
                "requested-count": 640,
                "requested-instances": [
                    {
                        "duration-millis": 3663,
                        "reason": {
                            "request": {
                                "request-method": "put",
                                "route-id": "puppet-v3-report-/*/",
                                "uri": "/puppet/v3/report/hostname.example.com"
                            }
                        },
                        "time": 1448478371715
                    }
                ],
                "return-count": 638
            }
        }
    }
}
~~~

#### GET /status/v1/services/pe-master

Returns JSON containing information about the routes that agents use to connect to this server. As with all other metrics endpoints, you must query it at port 8140 and append the `level=debug` URL parameter.

**Query parameters**

No parameters are supported. Defaults to using the _critical_ status level.

**Response codes**

* 200 if and only if all services report a status of _running_
* 503 if any service's status is _unknown_ or _error_

**Response keys**

The response's `experimental/http-metrics` section contains a list of routes, each containing the following keys:

* **aggregate:** The total time Puppet Server spent processing requests for this route.
* **count:** The total number of requests Puppet Server processed for this route.
* **mean:** The average time Puppet Server spent on each request for this route, calculated by dividing the `aggregate` value by the `count`.
* **route-id:** The route being served. The request returns a route with the special `route-id` of "total", which represents the aggregate data for all requests along all routes.

Routes for Puppet Enterprise 2015.2.3 and newer agents are prefixed with `puppet-v3`, while Puppet Enterprise 3 agents' routes are not. For example, a PE 2015.3 `route-id` might be `puppet-v3-report-/*/`, while the equivalent PE 3 agent's `route-id` is `:environment-report-/*/`.

**Example response**

~~~ json
"pe-master": {
    {...},
    "status": {
        "experimental": {
            "http-metrics": [
                {
                    "aggregate": 70668,
                    "count": 234,
                    "mean": 302,
                    "route-id": "total"
                },
                {
                    "aggregate": 28613,
                    "count": 13,
                    "mean": 2201,
                    "route-id": "puppet-v3-catalog-/*/"
                },
                {...}
            ]
        }
    }
}
~~~

#### GET /status/v1/services/pe-puppet-profiler

Returns JSON containing statistics about catalog compilation. You can use this data to discover which functions or resources are consuming the most resources or are most frequently used. As with all other metrics endpoints, you must query it at port 8140 and append the `level=debug` URL parameter.

Puppet Server's profiler is enabled by default, but if it has been disabled, this endpoint's metrics are not available. Instead, the endpoint returns the same keys returned by a standard status endpoint request (see [JSON endpoints](#json-endpoints)) and an empty `status` key.

**Query parameters**

No parameters are supported. Defaults to using the _critical_ status level.

**Response codes**

* 200 if and only if all services report a status of _running_
* 503 if any service's status is _unknown_ or _error_

**Response keys**

If the profiler is enabled, the response returns two subsections in the `experimental` section:

* `experimental/function-metrics`, containing statistics about functions evaluated by Puppet Server when compiling catalogs.
* `experimental/resource-metrics`, containing statistics about resources declared in manifests compiled by Puppet Server.

Each function measured in the `function-metrics` section also has a **function** key containing the function's name, and each resource measured in the `resource-metrics` section has a **resource** key containing the resource's name.

The two sections otherwise share these keys:

* **aggregate:** The total time spent handling this function call or resource during catalog compilation.
* **count:** The number of times Puppet Server has called the function or instantiated the resource during catalog compilation.
* **mean:** The average time spent handling this function call or resource during catalog compilation, calculated by dividing the `aggregate` value by the `count`.

**Example response**

~~~ json
"pe-puppet-profiler": {
    {...},
    "status": {
        "experimental": {
            "function-metrics": [
                {
                    "aggregate": 1628,
                    "count": 407,
                    "function": "include",
                    "mean": 4
                },
                {...},
            "resource-metrics": [
                {
                    "aggregate": 3535,
                    "count": 5,
                    "mean": 707,
                    "resource": "Class[Puppet_enterprise::Profile::Console]"
                },
                {...},
            ]
        }
    }
}
~~~

## The metrics API

Puppet Enterprise includes an optional, disabled-by-default web endpoint for [Java Management Extension (JMX)](https://docs.oracle.com/javase/tutorial/jmx/index.html) metrics, namely [managed beans (MBeans)](https://docs.oracle.com/javase/tutorial/jmx/mbeans/).

> **Note:** These API endpoints are a [tech preview][]. The metrics described here are returned only when passing the `level=debug` URL parameter, and the structure of the returned data might change in future versions.
>
> To enable this endpoint, you must first set `puppet_enterprise::master::puppetserver::metrics_webservice_enabled: true` in Hiera.

### GET /metrics/v1/mbeans

This endpoint lists available MBeans.

**Response keys**

* The key is the name of a valid MBean.
* The value is a URI to use when requesting that MBean's attributes.

### POST /metrics/v1/mbeans

This endpoint retrieves requested MBean metrics.

**Query parameters**

The query doesn't require any parameters, but the request body must contain a
JSON object whose values are metric names, or a JSON array of metric names, or
a JSON string containing a single metric's name.

For a list of metric names, make a `GET` request to `/metrics/v1/mbeans`.

**Response keys**

The response format, though always JSON, depends on the request format:

* Requests with a JSON object return a JSON object where the values of
  the original object are transformed into the Mbeans' attributes for the
  metric names.
* Requests with a JSON array return a JSON array where the items of the
  original array are transformed into the Mbeans' attributes for the
  metric names.
* Requests with a JSON string return the a JSON object of the Mbean's
  attributes for the given metric name.

### GET /metrics/v1/mbeans/<name>

These endpoints report on a single metric.

**Query parameters**

The query doesn't require any parameters, but the endpoint itself must correspond
to one of the metrics returned by a `GET` request to `/metrics/v1/mbeans`.

**Response keys**

The endpoint's responses contain a JSON object mapping strings to values. The
keys and values returned in the response vary based on the specified metric.

**Example response**

Use `curl` from localhost to request data on MBean memory usage:

```
curl 'http://localhost:8080/metrics/v1/mbeans/java.lang:type=Memory'
```

The response should contain a JSON object representing the data:

``` json
{
  "ObjectPendingFinalizationCount" : 0,
  "HeapMemoryUsage" : {
    "committed" : 807403520,
    "init" : 268435456,
    "max" : 3817865216,
    "used" : 129257096
  },
  "NonHeapMemoryUsage" : {
    "committed" : 85590016,
    "init" : 24576000,
    "max" : 184549376,
    "used" : 85364904
  },
  "Verbose" : false,
  "ObjectName" : "java.lang:type=Memory"
}
```
