---
author: melissa amos <melissa.amos@puppet.com\>
---

# Installing agents

You can install Puppet Enterprise agents on \*nix, Windows, and macOS.

The master hosts a package repo used to install agents in your infrastructure.

## How PE package management works

The master hosts a package repo used to install agents in your infrastructure.

The PE package management repo is created during installation of the master and serves packages over HTTPS using the same port as the master \(8140\). This means agents don't require any new ports to be open other than the one they already need to communicate with the master.

When you run the install script, it detects the OS on which it is running, sets up an apt, yum, or zipper repo that refers back to the master, and then pulls down and installs the `puppet-agent` packages. It also creates a basic `puppet.conf`, and kicks off a Puppet run.

If the install script can't find agent packages corresponding to the agent's platform, it fails with an error telling you which `pe_repo` class you need to add to the master.

Automatic downloading of agent installer packages using the `pe_repo` class requires an internet connection.

**Tip:** If your master uses a proxy server to access the internet, specify `puppet_enterprise::http_proxy_host` and `puppet_enterprise::http_proxy_port` in `pe.conf` prior to installation.

## How to use the install script  

The install script installs and configures the agent on target nodes, including automatically pointing agents to your master. You can pass additional configuration options to further customize the agent.

The install script is used in these installation methods:

-   [Install \*nix agents with PE package management](installing_agents.md#)

-   [Install \*nix agents using a manually transferred certificate](installing_agents.md#)

-   [Install \*nix agents with PE package management without internet access](installing_agents.md#)

-   [Install Windows agents with PE package management](installing_agents.md#)

-   [Install Windows agents using a manually transferred certificate](installing_agents.md#)


You can provide as flags to the script any number of these options, in any order, for example:

|Option|Example|Result|
|------|-------|------|
|`puppet.conf` settings|```
agent:splay=true
agent:certname=node1.corp.net
agent:environment=development
```

|The `puppet.conf` file looks like this: ```
[agent]
certname = node1.corp.net
splay = true
environment = development
```

|
|CSR attribute settings|```
extension_requests:pp_role=webserver
custom_attributes:challengePassword=abc123
```

|The installer creates a `csr_attributes.yaml` file before installing with this content: ```
---
custom_attributes:
 challengePassword: abc123
extension_requests:
 pp_role: webserver
```

|
|MSI properties \(Windows only\)|```
-PuppetAgentAccountUser ‘pup_adm’
-PuppetAgentAccountPassword ‘secret’
```

|The Puppet service runs as pup\_adm with a password of secret.|
|Puppet service status|\*nix:

```
--puppet-service-ensure stopped
--puppet-service-enable false
```

Windows

```
-PuppetServiceEnsure stopped
-PuppetServiceEnable false
```

|The Puppet service is stopped and doesn't boot after installation. An initial Puppet run doesn't occur after installation.|

### `puppet.conf` settings

You can specify any agent configuration option using the install script. Configuration settings are added to `puppet.conf`.

These are the most commonly specified agent config options:

-   server

-   certname

-   environment

-   splay

-   splaylimit

-   noop


**Tip:** On Enterprise Linux systems, if you have a proxy between the agent and the master, you can specify `http_proxy_host` , for example `-s agent:http_proxy_host=<PROXY_FQDN>`.

See the [Configuration Reference](https://puppet.com/docs/puppet/5.3/configuration.html) for details.

### CSR attribute settings

These settings are added to `puppet.conf` and included in the `custom_attributes` and `extension_requests` sections of `csr_attributes.yaml`.

You can pass as many parameters as needed. Follow the `section:key=value` pattern and leave one space between parameters.

See the [csr\_attributes.yaml](https://puppet.com/docs/puppet/5.3/config_file_csr_attributes.html) reference for details.

\*nix install script with example agent setup and certificate signing parameters:

```
curl -k https://master.example.com:8140/packages/current/install.bash | sudo bash -s agent:certname=<certnameOtherThanFQDN> custom_attributes:challengePassword=<passwordForAutosignerScript> extension_requests:pp_role=<puppetNodeRole>
```

Windows install script with example agent setup and certificate signing parameters:

```
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}; $webClient = New-Object System.Net.WebClient; $webClient.DownloadFile('https://<PUPPET MASTER FQDN>:8140/packages/current/install.ps1', 'install.ps1'); .\install.ps1 agent:certname=<certnameOtherThanFQDN> custom_attributes:challengePassword=<passwordForAutosignerScript> extension_requests:pp_role=<puppetNodeRole>
```

### MSI properties \(Windows\)

For Windows, you can set these MSI properties, with or without additional agent configuration settings.

|MSI Property|PowerShell flag|
|------------|---------------|
|`INSTALLDIR`|`-InstallDir`|
|`PUPPET_AGENT_ACCOUNT_USER`|`-PuppetAgentAccountUser`|
|`PUPPET_AGENT_ACCOUNT_PASSWORD`|`-PuppetAgentAccountPassword`|
|`PUPPET_AGENT_ACCOUNT_DOMAIN`|`-PuppetAgentAccountDomain`|

Windows install script with MSI properties and agent configuration settings: 

```
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}; $webClient = New-Object System.Net.WebClient; $webClient.DownloadFile('https://<MASTER HOSTNAME>:8140/packages/current/install.ps1', 'install.ps1'); .\install.ps1 -PuppetAgentAccountUser "svcPuppet" -PuppetAgentAccountPassword "s3kr3t_P@ssword" agent:splay=true agent:environment=development
```

### Puppet service status

By default, the install script starts the Puppet agent service and kicks off a Puppet run. If you want to manually trigger a Puppet run, or you're using a provisioning system that requires non-default behavior, you can control whether the service is running and enabled.  

|Option|\*nix|Windows|Values|
|------|-----|-------|------|
|`ensure`|`--puppet-service-ensure <VALUE>`|`-PuppetServiceEnsure <VALUE>`|-   running

-   stopped


|
|`enable`|`--puppet-service-enable <VALUE>`|`-PuppetServiceEnable <VALUE>`|-   true

-   false

-   manual \(Windows only\)

-   mask


|

## Installing \*nix agents

PE has package management tools to help you easily install and configure agents. You can also use standard \*nix package management tools.

### Install \*nix agents with PE package management

PE provides its own package management to help you install agents in your infrastructure.

#### About this task

The steps for agent installation differ based on whether the target machine has the same operating system and architecture as the master.

**Note:** The `<MASTER HOSTNAME>` portion of the installer script—as provided in the following example—refers to the FQDN of the master. The FQDN must be fully resolvable by the machine on which you're installing or upgrading the agent.

#### Install \*nix agents with same OS and architecture as the master

If the master and node on which you're installing the agent have the same OS and architecture, you can install agents without any additional configuration on your master.

##### About this task

##### Procedure

1.  SSH into the node where you want to install the agent, and run the installation command appropriate to your environment.

    -   Curl

        ```
        curl -k https://<MASTER HOSTNAME>:8140/packages/current/install.bash | sudo bash
        ```

        **Tip:** On AIX, which doesn't support the `-k` option, use `--tlsv1` instead. On other platforms that don't support the `-k` option, you must install using a manually transferred certificate.

    -   wget

        ```
        wget -O - -q --no-check-certificate --secure-protocol=TLSv1 https://<MASTER HOSTNAME>:8140/packages/current/install.bash | sudo bash
        ```

    -   Solaris 10 \(run as root\)

        ```
        export PATH=$PATH:/opt/sfw/bin
        wget -O - -q --no-check-certificate --secure-protocol=TLSv1 https://<MASTER HOSTNAME>:8140/packages/current/install.bash | bash
        ```

2.  Sign the agent's certificate.


#### Install \*nix agents with a different OS and architecture than the master

To install an agent with a different OS than the master, you add the appropriate class for the repo that contains the agent packages and then classify the **PE Master** node group with that class. You then run the install script from the agent to retrieve the necessary packages for installation.

##### About this task

In the following example, the master is on a node running RHEL 6 and the agent is on a node running Debian 6 on AMD64 hardware.

##### Procedure

1.  In the console, click **Classification**, and in the **PE Infrastructure** group, select the **PE Master** group.

2.  On the **Configuration** tab in the **Class name** field, enter `pe_repo` and select the repo class from the list of classes.

    **Note:** The repo classes are listed as `pe_repo::platform::<AGENT_OS_VERSION_ARCHITECTURE>`.

3.  Click **Add class**, and commit changes.

4.  Run `puppet agent -t` to configure the master node using the newly assigned class.

    The new repo is created in `/opt/puppetlabs/server/data/packages/public/<PE VERSION>/<PLATFORM>/`.

5.  SSH into the node where you want to install the agent, and run the installation command appropriate to your environment.

    -   Curl

        ```
        curl -k https://<MASTER HOSTNAME>:8140/packages/current/install.bash | sudo bash
        ```

        **Tip:** On AIX, which doesn't support the `-k` option, use `--tlsv1` instead. On other platforms that don't support the `-k` option, you must install using a manually transferred certificate.

    -   wget

        ```
        wget -O - -q --no-check-certificate --secure-protocol=TLSv1 https://<MASTER HOSTNAME>:8140/packages/current/install.bash | sudo bash
        ```

    -   Solaris 10 \(run as root\)

        ```
        export PATH=$PATH:/opt/sfw/bin
        wget -O - -q --no-check-certificate --secure-protocol=TLSv1 https://<MASTER HOSTNAME>:8140/packages/current/install.bash | bash
        ```

6.  Sign the agent's certificate.


### Install \*nix agents with your own package management

If you choose not to use PE package management to install agents, you can use your own package management tools.

#### Before you begin

[Download](https://puppet.com/misc/pe-files/pe_repo) the appropriate agent package.

#### About this task

Agent packages can be found on the master in `/opt/puppetlabs/server/data/packages/public/<PE VERSION>/`. This directory contains the platform specific repository file structure for agent packages. For example, if your master is running on CentOS 7, in `/opt/puppetlabs/server/data/packages/public/<PE VERSION>/`, there's a directory `el-7-x86_64`, which contains the directories with all the packages needed to install an agent.

If your nodes are running an operating system or architecture that is different from the master, download the appropriate agent package, extract the agent packages into the appropriate repo, and then install the agents on your nodes just as you would any other package, for example `yum install puppet-agent`.

#### Procedure

1.  Add the agent package to your own package management and distribution system.

2.  Configure the package manager on your agent node \(Yum, Apt\) to point to that repo.

3.  Install the agent using the command appropriate to your environment.

    -   Yum

        ```
        sudo yum install puppet-agent
        ```

    -   Apt

        ```
        sudo apt-get install puppet-agent
        ```

4.  Configure the agent as needed: `puppet config set`


### Install \*nix agents using a manually transferred certificate

If you choose not to or can't use `curl -k` to trust the master during agent installation, you can manually transfer the master CA certificate to any machines you want to install agents on, and then run the installation script against that cert.

#### About this task

**Note:** `-k` is not supported for AIX 5.3, 6.1, and 7.1. You must replace the `-k` flag with `-tlsv1` or `-1`.

#### Procedure

1.  On the machine that you're installing the agent on, create the directory `/etc/puppetlabs/puppet/ssl/certs`.

2.  On the master, navigate to `/etc/puppetlabs/puppet/ssl/certs/` and transfer `ca.pem` to the certs directory you created on the agent node. 

3.  On the agent node, verify file permissions: `chmod 444 /etc/puppetlabs/puppet/ssl/certs/ca.pem`

4.  Run the installation command, using the `--cacert` flag to point to the cert:

    ```
       curl --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem https://<MASTER HOSTNAME>:8140/packages/current/install.bash | sudo bash
    ```

5.  Sign the agent's certificate.


### Install \*nix agents without internet access

If you don't have access to the internet beyond your infrastructure, you can download the appropriate agent package from an internet-connected system and then install using the package management solution of your choice.

#### Before you begin

[Download](https://puppet.com/misc/pe-files/pe_repo) the appropriate agent package.

#### Install \*nix agents with PE package management without internet access

Use PE package management to install agents when you don't have internet access beyond your infrastructure.

##### About this task

**Note:** You must repeat this process each time you upgrade your master.

##### Procedure

1.  On your master, copy the agent package to `/opt/puppetlabs/server/data/staging/pe_repo-<PUPPET AGENT VERSION>`.

2.  Follow the steps for [Install \*nix agents with PE package management](installing_agents.md#).


#### Install \*nix agents with your own package management without internet access

Use your own package management to install agents when you don't have internet access beyond your infrastructure.

##### About this task

**Note:** You must repeat this process each time you upgrade your master.

##### Procedure

1.  Add the agent package to your own package management and distribution system.

2.  Disable the PE-hosted repo.

    1.  In the console, click **Classification**, and in the **PE Infrastructure** group, select the **PE Master** group.

    2.  On the **Configuration** tab, find `pe_repo` class \(as well as any class that begins `pe_repo::`\), and click **Remove this class**.

    3.  Commit changes.


#### Install \*nix agents from compile masters using your own package management without internet access

If your infrastructure relies on compile masters to install agents, you don’t have to copy the agent package to each compile master. Instead, use the console to specify a path to the agent package on your package management server.

##### About this task

##### Procedure

1.  Add the agent package to your own package management and distribution system.

2.  Set the `base_path` parameter of the `pe_repo` class to your package management server. 

    1.  In the console, click **Classification**, and in the **PE Infrastructure** group, select the **PE Master** group.

    2.  On the **Configuration** tab, find the `pe_repo` class and specify the parameter.

        |**Parameter**|**Value**|
        |-------------|---------|
        |**base\_path**|FQDN of your package management server.|

    3.  Click **Add parameter** and commit changes.


## Installing Windows agents

You can install Windows agents with PE package management, with a manually transferred certificate, or with the Windows .msi package.

### Install Windows agents with PE package management

To install a Windows agent with PE package management, you use the `pe_repo` class to distribute an installation package to agents. You can use this method with or without internet access.

#### Before you begin

If your master doesn't have internet access, [download](https://puppet.com/misc/pe-files/pe_repo) the appropriate agent package and save it on your master in the location appropriate for your agent systems:

-   32-bit systems — `/opt/puppetlabs/server/data/packages/public/<PE_VERSION>/windows-i386-<AGENT_VERSION>/`

-   64-bit systems — `/opt/puppetlabs/server/data/packages/public/<PE_VERSION>/windows-x86_64-<AGENT_VERSION>/`


#### About this task

You must use PowerShell 2.0 or later to install Windows agents with PE package management.

**Note:** The `<MASTER HOSTNAME>` portion of the installer script—as provided in the following example—refers to the FQDN of the master. The FQDN must be fully resolvable by the machine on which you're installing or upgrading the agent.

#### Procedure

1.  In the console, click **Classification**, and in the **PE Infrastructure** group, select the **PE Master** group.

2.  On the **Configuration** tab in the **Class name** field, select **pe\_repo** and select the appropriate repo class from the list of classes.

    -   64-bit \(x86\_64\) — **pe\_repo::platform::windows\_x86\_64**.
    -   32-bit \(i386\) — **pe\_repo::platform::windows\_i386**.
3.  Click **Add class** and commit changes.

4.  On the master, run Puppet to configure the newly assigned class.

    The new repository is created on the master at `/opt/puppetlabs/server/data/packages/public/<PE VERSION>/<PLATFORM>/`.

5.  On the node, open an administrative PowerShell window, and install:

    ```
    [Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}; $webClient = New-Object System.Net.WebClient; $webClient.DownloadFile('https://<MASTER HOSTNAME>:8140/packages/current/install.ps1', 'install.ps1'); .\install.ps1
    ```


#### Results

After running the installer, you see the following output, which indicates the agent was successfully installed.

```Puppet
Notice: /Service[puppet]/ensure: ensure changed 'stopped' to 'running'
service { 'puppet':
  ensure => 'running',
  enable => 'true',
}
```

### Install Windows agents using a manually transferred certificate

If you need to perform a secure installation on Windows nodes, you can manually transfer the master CA certificate to target nodes, and run a specialized installation script against that cert.

#### About this task

#### Procedure

1.  Transfer the installation script and the CA certificate from your master to the node you're installing.

    |File|Location on master|Location on target node|
    |----|------------------|-----------------------|
    |Installation script \(`install.ps1`\)|`/opt/puppetlabs/server/data/packages/public/`|Any accessible local directory.|
    |CA certificate \(`ca.pem`\)|`/etc/puppetlabs/puppet/ssl/certs/`|`C:\ProgramData\PuppetLabs\puppet\etc\ssl\certs\`|

2.  Run the installation script, using the `-UsePuppetCA` flag: `.\install.ps1 -UsePuppetCA`


### Install Windows agents with the .msi package

Use the Windows .msi package if you need to specify agent configuration details during installation, or if you need to install Windows agents locally without internet access.

#### Before you begin

[Download](https://puppet.com/download-puppet-enterprise) the .msi package.

**Tip:** To install on nodes that don't have internet access, save the .msi package to the appropriate location for your system:

-   32-bit systems — `/opt/puppetlabs/server/data/packages/public/<PE_VERSION>/windows-i386-<AGENT_VERSION>/`

-   64-bit systems — `/opt/puppetlabs/server/data/packages/public/<PE_VERSION>/windows-x86_64-<AGENT_VERSION>/`


#### Install Windows agents with the installer

Use the MSI installer for a more automated installation process. The installer can configure `puppet.conf`, create CSR attributes, and configure the agent to talk to your master.

##### Procedure

1.  Run the installer as administrator.

2.  When prompted, provide the hostname of your master, for example puppet.

3.  Sign the agent's certificate.


#### Install Windows agents using `msiexec` from the command line

Install the MSI manually from the the command line if you need to customize `puppet.conf`, CSR attributes, or certain agent properties.

##### Procedure

1.  On the command line of the node that you want to install the agent on, run the install command:

    ```
    msiexec /qn /norestart /i puppet-agent-<VERSION>-<ARCH>.msi
    ```

    **Tip:** You can specify `/l*v install.txt` to log the progress of the installation to a file.


#### MSI properties

If you install Windows agents from the command line using the .msi package, you can optionally specify these properties.

**Important:** If you set a non-default value for `PUPPET_MASTER_SERVER`, `PUPPET_CA_SERVER`, `PUPPET_AGENT_CERTNAME`, or `PUPPET_AGENT_ENVIRONMENT`, the installer replaces the existing value in `puppet.conf` and re-uses the value at upgrade unless you specify a new value. Therefore, if you've used these properties once, don't change the setting directly in `puppet.conf`; instead, re-run the installer and set a new value at installation.

|Property|Definition|Setting in `pe.conf`|Default|
|--------|----------|--------------------|-------|
|`INSTALLDIR`|Location to install Puppet and its dependencies.|n/a|-   32-bit — `C:\Program Files\Puppet Labs\Puppet`

-   64-bit — `C:\Program Files \Puppet Labs\Puppet`


|
|`PUPPET_MASTER_SERVER`|Hostname where the master can be reached.|`server`|`puppet`|
|`PUPPET_CA_SERVER`|Hostname where the CA master can be reached, if you're using multiple masters and only one of them is acting as the CA.|`ca_server`|Value of `PUPPET_MASTER_SERVER`|
|`PUPPET_AGENT_CERTNAME`|Node's certificate name, and the name it uses when requesting catalogs.

For best compatibility, limit the value of `certname` to lowercase letters, numbers, periods, underscores, and dashes.

|`certname`|Value of `facter fdqn`|
|`PUPPET_AGENT_ENVIRONMENT`|Node's environment. **Note:** If a value for the `environment` variable already exists in `puppet.conf`, specifying it during installation does not override that value.

|`environment`|`production`|
|`PUPPET_AGENT_STARTUP_MODE`|Whether and how the agent service is allowed to run. Allowed values are: -   `Automatic` — Agent starts up when Windows starts and remains running in the background.
-   `Manual` — Agent can be started in the services console or with `net start` on the command line.
-   `Disabled` — Agent is installed but disabled. You must change its startup type in the services console before you can start the service.

|n/a|`Automatic`|
|`PUPPET_AGENT_ACCOUNT_USER`|Windows user account the agent service uses. This property is useful if the agent needs to access files on UNC shares, because the default `LocalService` account can't access these network resources.

The user account must already exist**,** and may be a local or domain user. The installer allows domain users even if they have not accessed the machine before. The installer grants `Logon as Service` to the user, and if the user isn't already a local administrator, the installer adds it to the `Administrators` group.

This property must be combined with `PUPPET_AGENT_ACCOUNT_PASSWORD` and `PUPPET_AGENT_ACCOUNT_DOMAIN`.

|n/a|`LocalSystem`|
|`PUPPET_AGENT_ACCOUNT_PASSWORD`|Password for the agent's user account.|n/a|No Value|
|`PUPPET_AGENT_ACCOUNT_DOMAIN`|Domain of the agent's user account.|n/a|`.`|
|`REINSTALLMODE`|A [default MSI property](https://msdn.microsoft.com/en-us/library/windows/desktop/aa371182(v=vs.85).aspx) used to control the behavior of file copies during installation.**Important:** If you need to downgrade agents, use `REINSTALLMODE=amus` when calling `msiexec.exe` at the command line to prevent removing files that the application needs.

|n/a|amus as of puppet-agent 1.10.10 and puppet-agent 5.3.4

omus in prior releases

|

To install the agent with the master at `puppet.acme.com`:

```
msiexec /qn /norestart /i puppet.msi PUPPET_MASTER_SERVER=puppet.acme.com
```

To install the agent to a domain user `ExampleCorp\bob`:

```
msiexec /qn /norestart /i puppet-<VERSION>.msi PUPPET_AGENT_ACCOUNT_DOMAIN=ExampleCorp PUPPET_AGENT_ACCOUNT_USER=bob PUPPET_AGENT_ACCOUNT_PASSWORD=password
```

### Windows agent installation details

Windows nodes can fetch configurations from a master and apply manifests locally, and respond to orchestration commands.

After installing a Windows node, the **Start Menu** contains a **Puppet** folder with shortcuts for running the agent manually, running Facter, and opening a command prompt for use with Puppet tools.

**Note:** You must run Puppet with elevated privileges. Select **Run as administrator** when opening the command prompt.

The agent runs as a Windows service. By default, the agent fetches and applies configurations every 30 minutes. After the first Puppet run, the MCollective service starts and the node can be controlled with MCollective. The agent service and the MCollective service can be started and stopped independently using either the service control manager UI or the command line `sc.exe` utility.

Puppet is automatically added to the machine's `PATH` environment variable, so you can open any command line and run `puppet`, `facter` and the other batch files that are in the `bin` directory of the Puppet installation. Items necessary for the Puppet environment are also added to the shell, but only for the duration of execution of each of the particular commands.

The installer includes Ruby, Gems, Facter, and MCollective. If you have existing copies of these applications, such as Ruby, they aren't affected by the re-distributed version included with Puppet.

#### Program directory

Unless overridden during installation, PE and its dependencies are installed in `Program Files` at `\Puppet Labs\Puppet`.

You can locate the `Program Files` directory using the `PROGRAMFILES` variable or the `PROGRAMFILES(X86)` variable.

The program directory contains these subdirectories.

|Subdirectory|Contents|
|------------|--------|
|`bin`|scripts for running Puppet and Facter|
|`facter`|Facter source|
|`hiera`|Hiera source|
|`mcollective`|MCollective source|
|`misc`|resources|
|`puppet`|Puppet source|
|`service`|code to run the agent as a service|
|`sys`|Ruby and other tools|

#### Data directory

PE stores settings, manifests, and generated data — such as logs and catalogs — in the data directory. The data directory contains two subdirectories for the various components:

-   `etc` \(the `$confdir`\): Contains configuration files, manifests, certificates, and other important files.
-   `var` \(the `$vardir`\): Contains generated data and logs.

When you run Puppet with elevated privileges as intended, the data directory is located in the `COMMON_APPDATA.aspx` folder. This folder is typically located at `C:\ProgramData\PuppetLabs\`. Because the common app data directory is a system folder, it is hidden by default.

If you run Puppet without elevated privileges, it uses a `.puppet` directory in the current user's home folder as its data directory, which can result in unexpected settings.

## Installing macOS agents

You can install macOS agents with PE package management, from Finder, or from the command line.

To install macOS agents with PE package management, follow the steps to [Install \*nix agents with PE package management](installing_agents.md#).

**Important:** For macOS agents, the certname is derived from the name of the machine \( My-Example-Mac\). To prevent installation issues, make sure the name of the node uses lowercases letters. If you don’t want to change your computer’s name, you can enter the agent certname in all lowercase letters when prompted by the installer.

### Install macOS agents from Finder

You can use Finder to install the agent on your macOS machine.

#### Procedure

1.  Open the agent package .dmg and click the installer .pkg.

2.  Follow prompts in the installer dialog.

    You must include the master hostname and the agent certname.

    **Tip:** You can get the master hostname with `puppet cert list --all`.


### Install macOS agents from the command line

You can use the command line to install the agent on a macOS machine.

#### Procedure

1.  SSH into the node as a root or sudo user.

2.  Mount the disk image: `sudo hdiutil mount <DMGFILE>`

    A line appears ending with `/Volumes/puppet-agent-VERSION`. This directory location is the mount point for the virtual volume created from the disk image.

3.  Change to the directory indicated as the mount point in the previous step, for example: `cd /Volumes/puppet-agent-VERSION`

4.  Install the agent package: `sudo installer -pkg puppet-agent-installer.pkg -target /`

5.  Verify the installation: `/opt/puppetlabs/bin/puppet --version`

6.  Configure the agent to connect to the master: `/opt/puppetlabs/bin/puppet config set server <MASTER_HOSTNAME>`

7.  Configure the agent certname: ``/opt/puppetlabs/bin/puppet config set certname` <AGENT_CERTNAME>`


### macOS agent installation details

macOS agents include core Puppet functionality, plus platform-specific capabilities like package installation, service management with LaunchD, facts inventory with the System Profiler, and directory services integration.

## Installing non-root agents

Running agents without root privileges can assist teams using PE to work autonomously.

For example, your infrastructure’s platform might be maintained by one team with root privileges while your infrastructure’s applications are managed by a separate team \(or teams\) with diminished privileges. If the application team wants to be able to manage its part of the infrastructure independently, they can run Puppet without root privileges.

PE is installed with root privileges, so you need a root user to install and configure non-root access to a monolithic master. The root user who performs this installation can then set up non-root users on the master and any nodes running an agent.

Non-root users can perform a reduced set of management tasks, including configuring settings, configuring Facter external facts, running `puppet agent --test`, and running Puppet with non-privileged cron jobs or a similar scheduling service. Non-root users can also classify nodes by writing or editing manifests in the directories where they have write privileges.

### Install non-root \*nix agents

#### Before you begin

You must have a monolithic installation, and because non-root users can't use MCollective capabilities to manage nodes, MCollective must be disabled. \(In the console, in the **PE MCollective** group, on the **Rules** tab, remove the **aio\_agent\_version** fact.\)

#### About this task

**Note:** Unless specified otherwise, perform these steps as a root user.

#### Procedure

1.  Install the agent on each node that you want to operate as a non-root user.

2.  Log in to the agent node and add the non-root user:

    ```
    puppet resource user <UNIQUE NON-ROOT USERNAME> ensure=present managehome=true
    ```

    **Note:** Each non-root user must have a unique name.

3.  Set the non-root user password.

    For example, on most \*nix systems: `passwd <USERNAME>`

4.  Stop the `puppet` service:

    ```
    puppet resource service puppet ensure=stopped enable=false
    ```

    By default, the `puppet` service runs automatically as a root user, so it must be disabled.

5.  Disable the MCollective service:

    ```
    puppet resource service mcollective ensure=stopped enable=false
    ```

6.  Disable the Puppet Execution Protocol agent.

    1.  In the console, click **Classification**, and in the **PE Infrastructure** group, select the **PE Agent** group.

    2.  On the **Configuration** tab, select the **puppet\_enterprise::profile::agent** class, specify parameters, click **Add parameter**, and then commit changes. 

        |Parameter|Value|
        |---------|-----|
        |**pxp\_enabled**|false|

7.  Change to the non-root user and generate a certificate signing request:

    ```
    puppet agent -t --certname "<UNIQUE NON-ROOT USERNAME.HOSTNAME>" --server "<MASTER HOSTNAME>"
    ```

    **Tip:** If you wish to use `su - <NON-ROOT USERNAME>` to switch between accounts, make sure to use the `-` \(`-l` in some unix variants\) argument so that full login privileges are correctly granted. Otherwise you might see permission denied errors when trying to apply a catalog.

8.  On the master or in the console, approve the certificate signing request.

9.  On the agent node as the non-root user, set the node's certname and the master hostname, and then run Puppet.

    ```
    puppet config set certname <UNIQUE NON-ROOT USERNAME.HOSTNAME> --section agent
    ```

    ```
    puppet config set server <PUPPET MASTER HOSTNAME> --section agent
    ```

    ```
    puppet agent -t
    ```

    The configuration specified in the catalog is applied to the node.

    **Tip:** If you see Facter facts being created in the non-root user’s home directory, you have successfully created a functional non-root agent.


#### Results

#### What to do next

To confirm that the non-root agent is correctly configured, verify that:

-   The agent can request certificates and apply the catalog from the master when a non-root user runs Puppet: `puppet agent -t`

-   The agent service is not running: `service puppet status`

-   The non-root user isn't listed in the console in the **PE MCollective** group.

-   Non-root users can collect existing facts by running `facter` on the agent, and they can define new, external facts.


### Install non-root Windows agents

#### Before you begin

You must have a monolithic installation, and because non-root users can't use MCollective capabilities to manage nodes, MCollective must be disabled. \(In the console, in the **PE MCollective** group, on the **Rules** tab, remove the **aio\_agent\_version** fact.\)

#### About this task

**Note:** Unless specified otherwise, perform these steps as a root user.

#### Procedure

1.  Install the agent on each node that you want to operate as a non-root user.

2.  Log in to the agent node and add the non-root user:

    ```
    puppet resource user <UNIQUE NON-ADMIN USERNAME> ensure=present managehome=true password="puppet" groups="Users"
    ```

    **Note:** Each non-root user must have a unique name.

3.  Stop the `puppet` service:

    ```
    puppet resource service puppet ensure=stopped enable=false
    ```

    By default, the `puppet` service runs automatically as a root user, so it must be disabled.

4.  Disable the MCollective service:

    ```
    puppet resource service mcollective ensure=stopped enable=false
    ```

5.  Change to the non-root user and generate a certificate signing request:

    ```
    puppet agent -t --certname "<UNIQUE NON-ADMIN USERNAME>" --server "<MASTER HOSTNAME>"
    ```

    **Important:** This Puppet run submits a cert request to the master and creates a `/.puppet` directory structure in the non-root user’s home directory. If this directory is not created automatically, you must manually create it before continuing.

6.  As the non-root user, create a configuration file at `%USERPROFILE%/.puppet/puppet.conf` to specify the agent certname and the hostname of the master:

    ```
    [main]
    certname = <UNIQUE NON-ADMIN USERNAME.hostname>
    server = <MASTER HOSTNAME>
    ```

7.  As the non-root user, submit a cert request: `puppet agent -t`.

8.  On the master or in the console, approve the certificate signing request.

    **Important:** It's possible to sign the root user certificate in order to allow that user to also manage the node. However, this introduces the possibility of unwanted behavior and security issues. For example, if your site.pp has no default node configuration, running the agent as non-admin could lead to unwanted node definitions getting generated using alt hostnames, a potential security issue. If you deploy this scenario, ensure the root and non-root users never try to manage the same resources, have clear-cut node definitions, ensure that classes scope correctly, and so forth.

9.  On the agent node as the non-root user, run Puppet: `puppet agent -t`.

    The configuration specified in the catalog is applied to the node.


### Non-root user functionality

Non-root users can only use a subset of functionality. Any operation that requires root privileges, such as installing system packages, can't be managed by a non-root agent.

#### \*nix non-root functionality

On \*nix systems, as non-root agent you may enforce these resource types:

-   `cron` \(only non-root cron jobs can be viewed or set\)
-   `exec` \(cannot run as another user or group\)
-   `file` \(only if the non-root user has read/write privileges\)
-   `notify`
-   `schedule`
-   `ssh_key`
-   `ssh_authorized_key`
-   `service`
-   `augeas`

**Note:** When running a cron job as non-root user, using the `-u` flag to set a user with root privileges causes the job to fail, resulting in this error message:

```
 Notice: /Stage[main]/Main/Node[nonrootuser]/Cron[illegal_action]/ensure: created must be privileged to use -u
```

You may also inspect these resource types \(use `puppet resource <resource type>`\):

-   `host`
-   `mount`
-   `package`

#### Windows non-root functionality

On Windows systems as non-admin user, you may enforce these types :

-   `exec`
-   `file`

**Note:** A non-root agent on Windows is extremely limited as compared to non-root \*nix. While you can use the above resources, you are limited on usage based on what the agent user has access to do \(which isn't much\). For instance, you can't create a file\\directory in `C:\Windows` unless your user has permission to do so.

You may also inspect these resource types \(use `puppet resource <resource type>`\):

-   `host`
-   `package`
-   `user`
-   `group`
-   `service`

## Managing certificate signing requests

When you install a new PE agent, the agent automatically submits a certificate signing request \(CSR\) to the master.

Certificate requests can be signed from the console or the command line. If DNS altnames are set up for agent nodes, you must use the command line interface to approve and reject node requests.

After approving a node request, the node doesn’t show up in the console until the next Puppet run, which can take up to 30 minutes. You can manually trigger a Puppet run if you want the node to appear immediately.

To accept or reject CSRs in the console or on the command line, you need the permission **Certificate requests: Accept and reject**. To manage certificate requests in the console, you also need the permission **Console: View**.

### Managing certificate signing requests in the console

The console displays a list of nodes on the **Unsigned certs** page that have submitted CSRs. You can approve or deny CSRs individually or in a batch.

If you use the **Accept All** or **Reject All** options, processing could take up to two seconds per request.

When using **Accept All** or **Reject All**, nodes are processed in batches. If you close the browser window or navigate to another website while processing is in progress, only the current batch is processed.

### Managing certificate signing requests on the command line

You can view, approve, and reject node requests using the command line.

To view pending node requests on the command line:

```

                `$ sudo puppet cert list`
            
```

To sign a pending request:

```

                `$ sudo puppet cert sign <name>`
            
```

To sign pending requests for nodes with DNS altnames:

```

                `$ sudo puppet cert sign (<HOSTNAME> or --all) --allow-dns-alt-names``
            
```

## Configuring agents

You can add additional configuration to agents by editing `/etc/puppetlabs/puppet/puppet.conf` directly, or by using the `puppet config set` sub-command, which edits `puppet.conf` automatically.

For example, to point the agent at a master called `master.example.com`, run `puppet config set server master.example.com`. This command adds the setting `server = puppetmaster.example.com` to the `[main]` section of `puppet.conf`.

To set the certname for the agent, run `/opt/puppetlabs/bin/puppet config set certname agent.example.com`.

