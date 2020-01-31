---
layout: default
title: "Preconfigured node groups"
canonical: "/pe/latest/console_classes_groups_preconfigured_groups.html"
---

[puppet]: ./puppet_overview.html
[lang_classes]: {{puppet}}/lang_classes.html
[learn]: /learning/
[forge]: http://forge.puppetlabs.com
[modules]: {{puppet}}/modules_fundamentals.html
[topscope]: {{puppet}}/lang_scope.html#top-scope
[environment]: /guides/environment.html

During installation, Puppet Enterprise (PE) automatically creates a number of preconfigured node groups that are used to manage your PE configuration. Some of these node groups come with required classification (classes and parameters) already added and some of them come with rules or pinned nodes. This page provides details of how each of these preconfigured node groups look when you do a new install of PE.

>**Note:** The PE infrastructure groups are used to configure PE. In general, you should not make any changes to these node groups. If you want to add custom classifications to your PE infrastructure nodes, we recommend that you create new child groups and apply your classification there.

## The all nodes node group

This node group is at the top of the hierarchy tree. All other node groups stem from this node group.

#### Classes
No default classes. You should generally avoid adding classes to this node group.

#### Matching nodes
All nodes.

#### Notes
* You cannot modify the preconfigured rule that matches all nodes.

## PE infrastructure-related node groups

These are the groups that PE uses to manage its own configuration. They should remain in these default states, unless you are adjusting parameters as directed by documentation or Support, or pinning new nodes for documented functions like creating compile masters. Use of these node groups is optional but highly recommended.

### The PE infrastructure node group

This node group is the parent to all of the infrastructure-related node groups listed in this section. It holds shared data that nodes matching this group's child node groups need to know. This includes the hostnames and ports of  various services (such as master and PuppetDB) and database info (except for passwords).

> **WARNING:** NEVER REMOVE THE PE INFRASTRUCTURE GROUP.
>
> Removal of the PE Infrastructure node group will disrupt communication between all of your PE Infrastructure nodes (Master, PuppetDB, and Console).
>
> It is very important to correctly configure the `puppet_enterprise` class in this node group. The parameters set in this class affect the behavior of all other preconfigured node groups on this page that use classes starting with `puppet_enterprise::profile`. Incorrect configuration of this class could potentially cause a PE service outage.
>
> If you need to set up these preconfigured node groups for any reason, you must configure the PE Infrastructure node group first before any other of the preconfigured node groups (except for the PE MCollective node groups). Failure to do so will disconnect the Masters, Database, and Console services, and manual intervention will be required to restore PE Infrastructure functionality.

#### Classes
 `puppet_enterprise` (sets the default parameters for child node groups)

> Parameters:
>
> * `mcollective_middleware_hosts = ["<YOUR HOST>"]` (This value must be an array, even if there is only one value, e.g. `mcollective_middleware_hosts = ["master.testing.net"]`)
> * `database_host = "<YOUR HOST>"`
> * `puppetdb_host = "<YOUR HOST>"`
> * `database_port = "<YOUR PORT NUMBER>"` (Only required if you have changed the port number from the default. The default port number is 5432.)
> * `database_ssl = true` (Set to `true` if you're using the PE-installed postgres, and `false` if you're using your own postgres.)
> * `puppet_master_host = "<YOUR HOST>"`
> * `certificate_authority_host = "<YOUR HOST>"`
> * `console_port =	"<YOUR PORT NUMBER>"` (Only required if you have changed the port number from the default. The default port number is 443.)
> * `puppetdb_database_name = "pe-puppetdb"`
> * `puppetdb_database_user = "pe-puppetdb"`
> * `puppetdb_port = "<YOUR PORT NUMBER>"` (Only required if you have changed the port number from the default. The default port number is 8081.)
> * `console_host =	"<YOUR HOST>"`
> * `pcp_broker_host = "<YOUR HOST>"`
>
> **Note:** In a monolithic install, `<YOUR HOST>` is your Puppet master's certname. You can find the certname with the `puppet config print certname` command. In a split install, `<YOUR HOST>` is the certname of the server on which you installed the component.


#### Matching nodes

Nodes are not pinned to this node group. The PE Infrastructure node group is the parent to other infrastructure node groups, such as PE Master, and is only used to set classification that all child node groups need to inherit. You should never pin nodes directly to this node group.

### The PE certificate authority node group

This node group is used to manage the certificate authority.

#### Classes
`puppet_enterprise::profile::certificate_authority` (manages the certificate authority on the first master node)

#### Matching nodes
On a new install, the Puppet master node is pinned to this node group.

#### Notes
You should not add additional nodes to this node group.

### The PE MCollective node group

This node group is used to enable PE's [MCollective engine](./mco_overview.html) on all matching nodes.

#### Classes
`puppet_enterprise::profile::mcollective::agent` (manages the MCollective server)

#### Matching nodes
All nodes.

#### Notes
You may have some nodes, such as non-root nodes or network devices, that **should not** have MCollective enabled. You can create a rule in this node group to exclude these nodes from MCollective.

### The PE master node group

This node group is used to manage Puppet masters and [add additional compile masters](./install_multimaster.html#step-3-classify-the-new-compile-master-node).

#### Classes
* `puppet_enterprise::profile::master` (manages the Puppet master service)
* `puppet_enterprise::profile::mcollective::peadmin` (manages the peadmin MCollective client)
* `puppet_enterprise::profile::master::mcollective` (manages keys used by MCollective)

#### Matching nodes
On a new install, the Puppet master node is pinned to this node group.

### The PE Orchestrator node group

This node group is used to manage the application orchestration service.

#### Classes
`puppet_enterprise::profile::orchestrator` (manages the application orchestration service)

#### Matching nodes

On a new install, the Puppet master node is pinned to this node group.

#### Notes
You should not add additional nodes to this node group.

### The PE PuppetDB node group

This node group is used to manage the database service.

#### Classes
`puppet_enterprise::profile::puppetdb` (manages the PuppetDB service)

#### Matching nodes
On a new install, the PuppetDB server node is pinned to this node group.

#### Notes
You should not add additional nodes to this node group.

### The PE console node group

This node group is used to manage the console.

#### Classes
* `puppet_enterprise::profile::console` (manages the console, node classifier and RBAC)
* `puppet_enterprise::license` (manages the PE license file for the status indicator)

#### Matching nodes
On a new install, the Console server node is pinned to this node group.

#### Notes
You should not add additional nodes to this node group.


### The PE ActiveMQ broker node group

This node group is used to manage the ActiveMQ broker and [add additional ActiveMQ brokers](./install_add_activemq.html#step-3-create-the-activemq-hub-group).

#### Classes
`puppet_enterprise::profile::amq::broker` (manages the ActiveMQ MCollective broker)

#### Matching nodes
On a new install, the Puppet master server node is pinned to this node group.


### The PE Agent node group

This node group is used to manage the configuration of Puppet agents.

#### Classes
`puppet_enterprise::profile::agent` (manages your PE agent configuration)

#### Matching nodes
All managed nodes are pinned to this node group by default.


### The PE Infrastructure Agent node group

This node group is a subset of the PE Agent node group used to manage infrastructure-specific overrides.

#### Classes
`puppet_enterprise::profile::agent` (manages your PE agent configuration)

#### Matching nodes
All nodes used to run your Puppet infrastructure and managed by the PE installer are pinned to this node group by default, including the master, PuppetDB, console, and compile masters.

You might want to manually pin to this group any additional nodes used to run your infrastructure, such as compile master load balancer nodes. Pinning a compile master load balancer node to this group allows it to receive its catalog from the master of masters, rather than the compile master, which helps ensure availability.

## Environment node groups

There are two preconfigured environment node groups:

- **Production Environment**
- **Agent-Specified Environment**

The two environment node groups are only used to set environments. They should not contain any classification. See the [environments workflow page](./console_classes_groups_environment_override.html) for more information about working with environments.

These node groups have the **This is an environment group** option selected, which forces all matching nodes into the group's environment, even if those nodes match another node group that specifies a different environment. We strongly encourage this workflow because it avoids the environment conflicts that can happen when you unintentionally have nodes that match multiple node groups with conflicting environments.

> Note: Always make sure that the **This is an environment group** checkbox remains selected in both of these environment node groups.

### The agent-specified environment node group

Normally, an environment specified by the node classifier overrides any environment set in a node's own puppet.conf file (the _agent-specified environment_). However, sometimes you might want to let an agent specify its own environment (in testing, for example).

The **agent-specified environment** node group forces the environment specified in a node's puppet.conf file, ignoring any environments that have been set by the node classifier. If you want to use the agent-specified environment for a node, you should pin the node to this node group, or create a rule in the node group that matches the node.

#### Classes

You should never add any classes to this group. This group should only be used to set the agent-specified environment for matching nodes.

#### Matching nodes

Create rules to match nodes that should be assigned the agent-specified environment. Alternatively, you can manually pin the nodes to the group.

By default, this group matches no nodes.

#### Notes

The **This is an environment group** option should be selected in the node group metadata section.

### The production environment node group

Nodes in this group will be assigned to the production environment.

#### Classes

You should never add any classes to this group. This group should only be used to set the production environment for matching nodes.

#### Matching nodes
This group comes with a rule that matches all nodes.

#### Notes

The **This is an environment group** option should be selected in the node group metadata section.


