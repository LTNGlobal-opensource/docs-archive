# Start installing Windows agents

When a node is managed by Puppet, it runs a Puppet agent application, commonly called an agent. In this section you install a Windows Puppet agent, which regularly pulls configuration catalogs from a Puppet master and applies them locally. These instructions include how to sign the agent certificate request in the console.

These instructions assume that you've installed PE.

## How does a Puppet agent work?

Agents ensure that resources in a node stay in their desired state.

Periodically, a Puppet agent sends facts to a Puppet master and requests a catalog. The master compiles the catalog using several sources of information, and returns the catalog to the agent.

Once it receives a catalog, the agent applies it by checking each resource the catalog describes. If it finds any resources that are not in their desired state, the agent makes any changes necessary to correct them. \(Or, in no-op mode, it reports on what changes would have been needed.\)

After applying the catalog, the agent submits a report to its master. Reports from all the agents are stored in PuppetDB and can be accessed in the PE console.

## Step 1: Install an agent on your Windows machine

To install a Windows agent with PE package management, you use the `pe_repo` class to distribute an installation package to agents. You can use this method with or without internet access.

### About this task

You must use PowerShell 2.0 or later to install Windows agents with PE package management.

**Note:** The `<MASTER HOSTNAME>` portion of the installer script—as provided in the following example—refers to the FQDN of the master. The FQDN must be fully resolvable by the machine on which you're installing or upgrading the agent.

### Procedure

1.  In the console, click **Classification**, and in the **PE Infrastructure** group, select the **PE Master** group.

2.  On the **Configuration** tab in the **Class name** field, select **pe\_repo** and select the appropriate repo class from the list of classes.

    -   64-bit \(x86\_64\) — **pe\_repo::platform::windows\_x86\_64**
    -   32-bit \(i386\) — **pe\_repo::platform::windows\_i386**
3.  Click **Add class** and commit changes.

4.  On the master, run Puppet to configure the newly assigned class.

    The new repository is created on the master at `/opt/puppetlabs/server/data/packages/public/<PE VERSION>/<PLATFORM>/`.

5.  On the node, open an administrative PowerShell window, and install:

    ```
    [Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}; $webClient = New-Object System.Net.WebClient; $webClient.DownloadFile('https://<MASTER HOSTNAME>:8140/packages/current/install.ps1', 'install.ps1'); .\install.ps1
    ```


### Results

After running the installer, the following output indicates the agent was successfully installed.

```
Notice: /Service[puppet]/ensure: ensure changed 'stopped' to 'running'service { 'puppet':  ensure => 'running',  enable => 'true',}
```

## Step 2: Approve the certificate request

During installation, the agent node contacts the Puppet master and requests a certificate. To add the node to the console and to start managing its configuration, you must approve its certificate request.

### About this task

### Procedure

1.  In the console, load a list of currently pending node requests by clicking **Unsigned certs**.

2.  Click the **Accept All** button to approve the request and add the node.


### Results

The Puppet agent can now retrieve configurations from the master the next time Puppet runs.

## Step 3: Test the Windows agent node

You can wait until Puppet runs automatically, or you can manually trigger Puppet runs.

### About this task

By default, the agent fetches configurations from the Puppet master every 30 minutes. You can configure this interval in the `puppet.conf` file with the `runinterval` setting. Alternatively, you can manually trigger a Puppet run from the command line at any time.

### Procedure

1.  To start a Puppet run, run `puppet agent -t` on the SSH client.

    This triggers a single Puppet run on the agent with verbose logging.

    Note the long string of log messages, ending with this message: Notice: Applied catalog in <N\> seconds.


### Results

You are now fully managing the agent node! It has checked in with the Puppet master for the first time and received its configuration info. It continues to check in and fetch new configurations every 30 minutes.

Next, learn how to configure your agents with Puppet code, beginning with pre-built chunks of Puppet code called modules.

