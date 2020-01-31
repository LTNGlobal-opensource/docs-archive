---
layout: default
title: "Classes endpoint"
canonical: "/pe/latest/nc_classes.html"
---

## GET /v1/classes

Retrieve a list of all classes known to the node classifier.

#### Response format

The response is a JSON array of class objects.
Each class object contains the following keys:

* `name`: the name of the class (a string).
* `environment`: the name of the environment that this class exists in.
                 Note that the same class can exist in different environments and can have different parameters in each environment.
* `parameters`: an object describing the parameters and default parameter values for the class.
                The keys of this object are the parameter names (strings). Each value is the default value for the associated parameter (which can be any legal JSON value). If the value is `null`, then the parameter is required.

Here is an example of a class object:

~~~ javascript
{
  "name": "apache",
  "environment": "production",
  "parameters": {
    "default_mods": true,
    "default_vhost": true,
    ...
  }
}
~~~

(Note that all other operations on classes require using the environment-specific endpoints below.)

#### Class list caching

This and all other endpoints on this page return the classes *currently known* to the node classifier, which retrieves them periodically from the Puppet master. To force an update, use the [`update_classes` endpoint](./nc_update_classes.html). To determine when classes were last retrieved from the Puppet master, use the [`last_class_update` endpoint](./nc_last_class_update.html).

## GET /v1/environments/\<environment\>/classes

Retrieve a list of all classes known to the node classifier within the given environment.

#### Response format

A JSON array of class objects as explained [above](#response-format).

#### Error responses

No error responses specific to this request are expected.

## GET /v1/environments/\<environment\>/classes/\<name\>

Retrieve the class with the given name in the given environment.

#### Response format

If the class exists, the response will be a class object as described [above](#response-format), in JSON format.

#### Error responses

If the class with the given name cannot be found, the server will return a 404 Not Found response with an empty body.
