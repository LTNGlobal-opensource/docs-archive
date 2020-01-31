---
layout: default
title: "Last class update endpoint"
canonical: "/pe/latest/nc_last_class_update.html"
---

## GET /v1/last-class-update

Retrieve the time that classes were last updated from the Puppet master.

#### Response

The response will always be an object with one field, `last_update`. If there has been an update the value of `last_update` will be the time of the last update in ISO8601 format. If the node classifier has never updated from Puppet it will be null.
