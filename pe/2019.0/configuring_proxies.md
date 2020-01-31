---
author: Melissa Amos <melissa.amos@puppet.com\>
---

# Configuring proxies

You can work around limited internet access by configuring proxies at various points in your infrastructure, depending on your connectivity limitations.

The examples provided here assume an unauthenticated proxy running at `proxy.example.vlan` on port 8080.

## Downloading agent installation packages through a proxy

If your master doesn't have internet access, it can't download agent installation packages. If you want to use package management to install agents, set up a proxy and specify its connection details so that `pe_repo` can access agent tarballs.

In `pe.conf`, Hiera, or the console, in the `pe_repo` class of the **PE Master** node group, specify values for `pe_repo::http_proxy_host` and `pe_repo::http_proxy_port` settings. For example, to specify these settings in `pe.conf`, add these lines:

```
"pe_repo::http_proxy_host": "proxy.example.vlan",
"pe_repo::http_proxy_port": 8080
```

**Tip:** To test proxy connections to `pe_repo`, run:

```
curl -x http://proxy.example.vlan:8080 -I https://pm.puppetlabs.com
```

## Setting a proxy for agent traffic

General proxy settings in `puppet.conf` manage HTTP connections that are directly initiated by the agent.

To configure agents to communicate through a proxy, specify values for the `http_proxy_host` and `http_proxy_port` settings in `/etc/puppetlabs/puppet/puppet.conf`, for example:

```
http_proxy_host = proxy.example.vlan
http_proxy_port = 8080
```

For more information about HTTP proxy host options, see the Puppet [configuration reference](https://puppet.com/docs/puppet/latest/configuration.html#httpproxyhost).

## Setting a proxy for Code Manager traffic

Code Manager has its own set of proxy configuration options which you can use to set a proxy for connections to the Git server or the Forge. These settings are unaffected by the proxy settings in `puppet.conf`, because Code Manager is run by Puppet Server.

**Note:** In order to set a proxy for Code Manager connections, you must use an HTTP URL for your r10k remote and for all Puppetfile module entries.

To configure Code Manager to use a proxy for all HTTP connections, including both Git and the Forge, do one of the following:

-   In the console, in the `puppet_enterprise::profile::master` class of the **PE Master** node group, specify a value for the `r10k_proxy` parameter.

-   In Hiera, add this key:

```
puppet_enterprise::profile::master::r10k_proxy: "http://proxy.example.vlan:8080"
```


**Tip:** To test proxy connections to Git or the Forge, run one of these commands:

```
curl -x http://proxy.example.vlan:8080 -I https://github.com
```

```
curl -x http://proxy.example.vlan:8080 -I https://forgeapi.puppet.com
```

For detailed information about configuring proxies for Code Manager traffic, see the Code Manager documentation.

**Related information**  


[Configuring proxies](code_mgr_customizing.md#)

