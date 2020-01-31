# Puppet orchestrator API: command endpoint

Use the `/command` endpoint to start and stop orchestrator jobs for tasks and plans.

## POST /command/deploy

Run the orchestrator across all nodes in an environment.

### Request format

The request body must be a JSON object using these keys:

|Key|Definition|
|---|----------|
|`environment`|The environment to deploy. **This key is required.**|
|`scope`|Object, required unless `target` is specified. The PuppetDB query, a list of nodes, a classifier node group id, or an application/application instance to deploy.|
|`description`|String, a description of the job.|
|`noop`|Boolean, whether to run the agent in no-op mode. The default is `false`.|
|`no_noop`|Boolean, whether to run the agent in enforcement mode. Defaults to `false`. This flag overrides `noop = true` if set in the agent's `puppet.conf`, and cannot be set to true at the same time as the `noop` flag.|
|`concurrency`|Integer, the maximum number of nodes to run at once. The default, if unspecified, is unlimited.|
|`enforce_environment`|Boolean, whether to force agents to run in the same environment in which their assigned applications are defined. This key is required to be `false` if `environment` is an empty string|
|`debug`|Boolean, whether to use the `--debug` flag on Puppet agent runs.|
|`trace`|Boolean, whether to use the `--trace` flag on Puppet agent runs.|
|`evaltrace`|Boolean, whether to use the `--evaltrace` flag on Puppet agent runs.|
|`target`|String,  required unless `scope` is specified. The name of an application or application instance to deploy. If an application is specified, all instances of that application are deployed. If this key is left blank or unspecified without a scope, the entire environment is deployed. **This key is deprecated.**|

For example, to deploy the `node1.example.com` environment in no-op mode, the following request is valid:

```json
{
  "environment" : "",
  "noop" : true,
  "scope" : {
    "nodes" : ["node1.example.com"]
  }
}
```

### Scope

Scope is a JSON object containing exactly one of these keys:

|Key|Definition|
|---|----------|
|`application`|The name of an application or application instance to deploy. If an application type is specified, all instances of that application are deployed.|
|`nodes`|A list of node names to target.|
|`query`|A PuppetDB or PQL query to use to discover nodes. The target is built from `certname` values collected at the top level of the query.|
|`node_group`|A classifier node group ID. The ID must correspond to a node group that has defined rules. It is not sufficient for parent groups of the node group in question to define rules. The user must also have permissions to view the node group.|

To deploy an application instance in the `production` environment:

```json
{
  "environment" : "production",
  "scope" : {
    "application" : "Wordpress_app[demo]"
  }
}
```

To deploy a list of nodes:

```json
{
  "environment" : "production",
  "scope" : {
    "nodes" : ["node1.example.com", "node2.example.com"]
  }
}
```

To deploy a list of nodes with the `certname` value matching a regex:

```json
{
  "environment" : "production",
  "scope" : {
    "query" : ["from", "nodes", ["~", "certname", ".*"]]
  }
}
```

To deploy to the nodes defined by the "All Nodes" node group:

```json
{
  "environment" : "production",
  "scope" : {
    "node_group" : "00000000-0000-4000-8000-000000000000"
  }
}
```

### Response format

If all node runs succeed, and the environment is successfully deployed, the server returns a `202` response.

The response is a JSON object containing a link to retrieve information about the status of the job and uses any one of these keys:

|Key|Definition|
|---|----------|
|`id`|An absolute URL that links to the newly created job.|
|`name`|The name of the newly created job.|

For example:

```json
{
  "job" : {
    "id" : "https://orchestrator.example.com:8143/orchestrator/v1/jobs/1234"
    "name" : "1234"
  }
}
```

### Error responses

For this endpoint, the `kind` key of the error displays the conflict.

|Key|Definition|
|---|----------|
|`puppetlabs.orchestrator/unknown-environment`|If the environment does not exist, the server returns a `404` response.|
|`puppetlabs.orchestrator/empty-environment`|If the environment requested contains no applications or no nodes, the server returns a `400` response.|
|`puppetlabs.orchestrator/empty-target`|If the application instance specified to deploy does not exist or is empty, the server returns a `400` response.|
|`puppetlabs.orchestrator/dependency-cycle`|If the application code contains a cycle, the server returns a `400` response.|
|`puppetlabs.orchestrator/puppetdb-error`|If the orchestrator is unable to make a query to PuppetDB, the server returns a `400` response.|
|`puppetlabs.orchestrator/query-error`|If a user does not have appropriate permissions to run a query, or if the query is invalid, the server returns a `400` response.|

**Related information**  


[Puppet orchestrator API: forming requests](orchestrator_api_forming_requests.md)

[Puppet orchestrator API: error responses](orchestrator_api_error_responses.md)

## POST /command/stop

Stop an orchestrator job that is currently in progress. Puppet agent runs that are in progress finish, but no new agent runs start. While agents are finishing, the server continues to produce events for the job.

The job transitions to status `stopped` when all pending agent runs have finished.

This command is *idempotent*: it can be issued against the same job any number of times.

### Request format

The JSON body of the request must contain the ID of the job to stop. The job ID is the same value as the `name` property returned with the `deploy` command.

-   `job`: the name of the job to stop.

For example:

```json
{
  "job": "1234"
}
```

### Response format

If the job is stopped successfully, the server returns a `202` response. The response uses these keys:

|Key|Definition|
|---|----------|
|`id`|An absolute URL that links to the newly created job.|
|`name`|The name of the newly created job.|
|`nodes`|A hash that shows all of the possible node statuses, and how many nodes are currently in that status.|

For example:

```json
{
  "job" : {
    "id" : "https://orchestrator.example.com:8143/orchestrator/v1/jobs/1234",
    "name" : "1234",
    "nodes" : {
      "new" : 5,
      "running" : 8,
      "failed"  : 3,
      "errored" : 1,
      "skipped" : 2,
      "finished": 5
    }
  }
}
```

### Error responses

For this endpoint, the `kind` key of the error displays the conflict.

|Key|Definition|
|---|----------|
|`puppetlabs.orchestrator/validation-error`|If a job name is not valid, the server returns a `400` response.|
|`puppetlabs.orchestrator/unknown-job`|If a job name is unknown, the server returns a `404` response.|

**Related information**  


[Puppet orchestrator API: error responses](orchestrator_api_error_responses.md)

## POST /command/task

Run a permitted task job across a set of nodes.

### Request format

The request body must be a JSON object and uses the following keys:

|Key|Definition|
|---|----------|
|`environment`|The environment to load the task from. The default is `production`.|
|`scope`|The PuppetDB query, list of nodes, or a node group ID. Application scopes are not allowed for task jobs. **This key is required.**|
|`description`|A description of the job.|
|`noop`|Whether to run the job in no-op mode. The default is `false`.|
|`task`|The task to run on the targets. **This key is required.**|
|`params`|The parameters to pass to the task. This key is required, but can be an empty object.|

For example, to run the `package` task on `node1.example.com`, the following request is valid:

```
{
  "environment" : "test-env-1",
  "task" : "package",
  "params" : {
    "action" : "install",
    "name" : "httpd"
  },
  "scope" : {
    "nodes" : ["node1.example.com"]
  }
}
```

### Scope

Scope is a JSON object containing exactly one of the following keys:

|Key|Definition|
|---|----------|
|`nodes`|An array of node names to target.|
|`query`|A PuppetDB or PQL query to use to discover nodes. The target is built from the `certname` values collected at the top level of the query.|
|`node_group`|A classifier node group ID. The ID must correspond to a node group that has defined rules. It is not sufficient for parent groups of the node group in question to define rules. The user must also have permissions to view the node group. Any nodes specified in the scope that the user does not have permissions to run the task on are excluded.|

To deploy an application instance in the production environment, the following request is valid:

```
{
  "environment" : "production",
  "scope" : {
    "application" : "Wordpress_app[demo]"
  }
}
```

To run a task on a list of nodes, the following request is valid:

```
{
  "environment" : "production",
  "scope" : {
    "nodes" : ["node1.example.com", "node2.example.com"]
  }
}
```

To run a task on a list of nodes with the `certname` value matching a regex, the following request is valid:

```
{
  "environment" : "production",
  "scope" : {
    "query" : ["from", "nodes", ["~", "certname", ".*"]]
  }
}
```

To deploy to the nodes defined by the "All Nodes" node group the following request is valid:

```
{
  "environment" : "production",
  "scope" : {
    "node_group" : "00000000-0000-4000-8000-000000000000"
  }
}
```

### Response format

If the task is starts successfully, the response will have a `202` status.

The response will be a JSON object containing a link to retrieve information about the status of the job. The keys of this object are:

-   `id`: an absolute URL that links to the newly created job.
-   `name`: the name of the newly created job. For example:

    ```
    {
      "job" : {
        "id" : "https://orchestrator.example.com:8143/orchestrator/v1/jobs/1234",
        "name" : "1234"
      }
    }
    ```


### Error responses

For this endpoint, the `kind` key of the error displays the conflict.

|Key|Definition|
|---|----------|
|`puppetlabs.orchestrator/unknown-environment`|If the environment does not exist, the server returns a 404 response.

|
|`puppetlabs.orchestrator/empty-target`|If the application instance specified to deploy does not exist or is empty, the server returns a 400 response.|
|`puppetlabs.orchestrator/puppetdb-error`|If the orchestrator is unable to make a query to PuppetDB, the server returns a 400 response.|
|`puppetlabs.orchestrator/query-error`|If a user does not have appropriate permissions to run a query, or if the query is invalid, the server returns a 400 response.|
|`puppetlabs.orchestrator/not-permitted`|This error occurs when a user does not have permission to run the task on the requested nodes. Server returns a 403 response.

|

**Related information**  


[Puppet orchestrator API: error responses](orchestrator_api_error_responses.md)

## POST /command/schedule\_task

Schedule a task to run at a future date and time.

### Request format

The request body must be a JSON object and uses the following keys:

|Key|Definition|
|---|----------|
|`task`|The task to run on the targets. Required.|
|`params`|The parameters to pass to the task. Required.|
|`scope`|The PuppetDB query, list of nodes, or a node group ID. Application scopes are not allowed for task jobs. Required.|
|`scheduled_time`|The ISO-8601 timestamp the determines when to run the scheduled job. If timestamp is in the past, a 400 error is thrown. Required.|
|`description`|A description of the job.|
|`environment`|The environment to load the task from. The default is `production`.|
|`noop`|Whether to run the job in no-op mode. The default is false.|

For example, to run the `package` task on `node1.example.com`, the following request is valid:

```
{
  "environment" : "test-env-1",
  "task" : "package",
  "params" : {
    "action" : "install",
    "package" : "httpd"
  },
  "scope" : {
    "nodes" : ["node1.example.com"]
  },
  "scheduled_time" : "2027-05-05T19:50:08Z"
}
```

### Response format

If the task schedules successfully the server returns 202.

The response is a JSON object containing a link to retrieve information about the status of the job. The keys of this object are

-   `id`: an absolute URL that links to the newly created job.
-   `name`: the name of the newly created job.

For example:

```
{
  "scheduled_job" : {
    "id" : "https://orchestrator.example.com:8143/orchestrator/v1/scheduled_jobs/2",
    "name" : "1234"
  }
}
```

### Error responses

For this endpoint, the `kind` key of the error displays the conflict.

|Key|Definition|
|---|----------|
|`puppetlabs.orchestrator/invalid-time`|If the `scheduled_time` provided is in the past, the server returns a 400 response.

|

**Related information**  


[Puppet orchestrator API: error responses](orchestrator_api_error_responses.md)

