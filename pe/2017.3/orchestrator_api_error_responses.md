# Puppet orchestrator API: error responses

From time to time, you may encounter an error using the orchestrator API. In such cases, you'll receive an error response.

Every error response from the Puppet orchestrator will be a JSON response. Each response will be an object containing the following keys:

|Key|Definition|
|---|----------|
|`kind`|The kind of error encountered.|
|`msg`|The message associated with the error.|
|`details`|A hash with more information about the error.|

For example, if an environment does not exist for a given request, an error is raised similar to the following:

```json
{
  "kind" : "puppetlabs.orchestrator/unknown-environment",
  "msg" : "Unknown environment doesnotexist",
  "details" : {
    "environment" : "doesnotexist"
  }
}
```

