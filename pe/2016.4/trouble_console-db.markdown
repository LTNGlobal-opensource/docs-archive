---
layout: default
title: "Troubleshooting the databases"
canonical: "/pe/latest/trouble_console-db.html"
---

Below are some common issues that can cause trouble with the databases that support the console.

**Note:** If you will be using your own instance of PostgreSQL (as opposed to the instance PE can install) for the console and PuppetDB, it must be version 9.1 or higher.


Using cURL to troubleshoot classification info in the PE console
--------


In past versions, you could run an external node script to reach the PE console node classifier (NC) to troubleshoot node and group classification information in the console. Due to changes in console authentication, that external node script was removed. However, you can now cURL the console to troubleshoot the NC. Consider the following examples:

#### Determine what node groups the NC has and what data they contain

Execute the following cURL command from the Puppet master (monolithic install) or from the PE console (split install):

~~~~
    curl https://$(hostname -f):4433/classifier-api/v1/groups > classifier_groups.json
      --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem
      --cert /etc/puppetlabs/puppet/ssl/certs/<WHITELISTED CERTNAME>.pem
      --key /etc/puppetlabs/puppet/ssl/private_keys/<WHITELISTED CERTNAME>.pem
~~~~

This will generate a file called `classifier_groups.json`. The JSON file is described in the [groups portion](./nc_groups.html#get-v1groups) of the NC API docs.

#### Determine what data the NC will generate for a given node name

**NOTE**: In the examples below replace `<SOME NODE NAME>` with the FQDN of the node you are interested in.

Execute the following cURL command from the Puppet master (monolithic install) or from the PE console (split install):

~~~~
	curl https://$(hostname -f):4433/classifier-api/v1/classified/nodes/<SOME NODE NAME> > node_classification.json
      --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem
      --cert /etc/puppetlabs/puppet/ssl/certs/<WHITELISTED CERTNAME>.pem
      --key /etc/puppetlabs/puppet/ssl/private_keys/<WHITELISTED CERTNAME>.pem
~~~~

This will generate a file called `node_classification.json`. The JSON file is described in the [classificatiopn portion](./nc_classification.html#post-v1classifiednodesname) of the NC API docs.

However, note that the above query will only return classification data for nodes that are [statically pinned](./console_classes_groups.html#adding-nodes-statically) to node groups.

To get classification data for [dynamically grouped nodes](./console_classes_groups.html#adding-nodes-dynamically), a JSON object containing facts will need to be submitted during the POST request.

~~~~
    curl https://$(hostname -f):4433/classifier-api/v1/groups > classifier_groups.json
      --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem
      --cert /etc/puppetlabs/puppet/ssl/certs/<WHITELISTED CERTNAME>.pem
      --key /etc/puppetlabs/puppet/ssl/private_keys/<WHITELISTED CERTNAME>.pem
~~~~

See the [classificatiopn portion](./nc_classification.html#post-v1classifiednodesname) of the NC API docs for more information on how to supply facts when making classification requests.

PostgreSQL is taking up too much space
-----

PostgreSQL should have `autovacuum=on` set by default. If you're having memory issues from the database growing too large and unwieldy, make sure this setting did not get turned off.

PostgreSQL buffer memory causes PE install to fail
-------

In some cases, when installing PE on machines with large amounts of RAM, the PostgreSQL database will use more shared buffer memory than is available and will not be able to start. This will prevent PE from installing correctly. The following error will be present in `/var/log/pe-postgresql/pgstartup.log`:

    FATAL: could not create shared memory segment: No space left on device
    DETAIL: Failed system call was shmget(key=5432001, size=34427584512,03600).

A suggested workaround is tweak the machine's `shmmax` and `shmall` kernel settings before installing PE. The `shmmax` setting should be set to approximately  50% of the total RAM; the `shmall` setting can be calculated by dividing the new `shmmax` setting by the PAGE_SIZE.  (`PAGE_SIZE` can be confirmed by running `getconf PAGE_SIZE`).

Use the following commands to set the new kernel settings:

    sysctl -w kernel.shmmax=<your shmmax calculation>
    sysctl -w kernel.shmall=<your shmall calculation>

Alternatively, you can also report the issue to the [Puppet customer support portal](https://support.puppetlabs.com/access/unauthenticated).

PuppetDB's default port conflicts with another service
-----

By default, PuppetDB communicates over port 8081. In some cases, this may conflict with existing services (e.g., McAfee's ePO). You can work around this issue by installing in text-mode with a parameter in `pe.conf` that specifies a different port with `puppet_enterprise::puppetdb_port`. For more information on installing in text-mode, refer the [documentation for text-mode installs](./install_text_mode.html)

`Puppet resource` generates Ruby errors after connecting `puppet apply` to PuppetDB
-----

Users who wish to use `puppet apply` (typically in deployments running masterless puppet), need to get it working with PuppetDB. If they do so by modifying `puppet.conf` to add the parameters `storeconfigs_backend = puppetdb` and `storeconfigs = true` in both the [main] and [master] sections), then `puppet resource` will cease to function and will display a Ruby run error. To avoid this, the correct way to get `puppet apply` connected to PuppetDB is to modify `/etc/puppetlabs/puppet/routes.yaml ` to correctly define the behavior of `puppet apply` without affecting other functions. The PuppetDB manual has [complete information and code samples]({{puppetdb}}/connect_puppet_apply.html).


