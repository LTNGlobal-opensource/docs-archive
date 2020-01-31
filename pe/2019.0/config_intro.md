# Methods for configuring Puppet Enterprise

After you've installed Puppet Enterprise \(PE\), optimize it for your environment by configuring and tuning settings as needed. For example, you might want to add your own certificate to the whitelist, increase the max-threads setting for `http` and `https` requests, or configure the number of JRuby instances.

There are three main methods for configuring PE: using the console, adding a key to Hiera, or editing `pe.conf`. For the most part, you can choose to use whatever method you want, but sometimes one might be better than another depending on the situation. In general, try to stay as consistent as possible.

**Important:** When you enable high availability, you must use Hiera or `pe.conf` only — not the console — to specify configuration parameters. Using `pe.conf` or Hiera ensures that configuration is applied to both your master and replica.

**Related information**  


[Configuring and tuning Puppet Server](config_puppetserver.md#)

[Configuring and tuning PuppetDB](config_puppetdb.md#)

[Configuring and tuning the console](config_console.md#)

[Configuring and tuning orchestration](config_orchestration.md#)

[Configuring Java arguments](config_java_args.md#)

## Configure settings using the console

Using the console to change parameters is the most user-friendly way of configuring Puppet Enterprise \(PE\) because you can use a graphical interface in order to make changes.

### About this task

Changes in the console will override your Hiera data and data in `pe.conf`. It is usually best to use the console when you want to:

-   Change parameters in profile classes starting with `puppet_enterprise::profile`.
-   Add any parameters in PE-managed configuration files.
-   Set parameters that configure at runtime.

To configure settings in the console, do the following:

### Procedure

1.  In the console, click **Classification**, and select the node group that contains the class you want to work with.

2.  On the **Configuration** tab, find the class you want to work with, select the **Parameter name** from the list and edit its value.

    If you wanted to change the number used to identify the port your console is on from the default 443 to 500, change the parameter value in the following:

    |Class|Parameter|Value|
    |-----|---------|-----|
    |`puppet_enterprise::profile::console`|`console_ssl_listen_port`|`[500]`|

3.  Click **Add parameter** and commit changes.

4.  On the nodes hosting the master and console, run Puppet.


**Related information**  


[Preconfigured node groups](preconfigured_node_groups.md#)

## Configure settings with Hiera

Hiera is a hierarchy-based method of configuration management that relies on a “defaults, with overrides” system. When you add a parameter or setting to your Hiera data, Hiera searches through the data in the order it was written to find the value you want to change. Once found, it overrides the default value with the new parameter or setting.

### Before you begin

For more information on how to use Hiera, see the [Hiera docs](https://puppet.com/docs/puppet/6.5/hiera.html).

### About this task

Changes in Hiera will override `pe.conf`, but not the console. It's best to use Hiera when you want to:

-   Change parameters in non-profile classes.
-   Set parameters that are static and version controlled.
-   Configure for high availability.

To configure a setting using Hiera, do the following:

### Procedure

1.  Open your default data file.

    The default location for Hiera data files is:

    -   \*nix: `/etc/puppetlabs/code/environments/<ENVIRONMENT>/data/common.yaml`

    -   Windows: `%CommonAppData%\PuppetLabs\code\environments\<ENVIRONMENT>\data\common.yaml`

    If you customize the `hiera.yaml` configuration to change location for data files \(the `datadir` setting\) or the path of the common data file \(in the `hierarchy` section\), look for the default `.yaml` file in the customized location.

2.  Add your new parameter to the file in editor.

    If you wanted to increase the number of seconds before a node is considered unresponsive from the default 3600 to 4000, add the following to your `.yaml` default file and insert your new parameter at the end.

    ```
    Puppet_enterprise::console_services::no_longer_reporting_cutoff: <4000>
    ```

3.  Compile and view your overrides.


**Related information**  


[Preconfigured node groups](preconfigured_node_groups.md#)

## Configure settings in `pe.conf`

Puppet Enterprise \(PE\) configuration data includes any data set in `/etc/puppetlabs/enterprise/conf.d/` but `pe.conf` is the file used for most configuration activities during installation.

### About this task

PE configuration settings made in Hiera and the console always override settings made in `pe.conf`. Configure settings using `pe.conf` when you want to:

-   Access settings during installation.
-   Configure for high availability.

To configure settings using `pe.conf`:

### Procedure

1.  Open the `pe.conf` file on your master:

    ```
    /etc/puppetlabs/enterprise/conf.d/pe.conf
    ```

2.  Add the parameter and new value you want to set.

    For example, to change the proxy in your repo, add the following and change the parameter to your new proxy location.

    ```
    pe_repo::http_proxy_host: "proxy.example.vlan"
    ```

3.  Run `puppet agent -t`

    **Note:** If PE services are stopped, run `puppet infrastructure configure` instead of `puppet agent -t`.


