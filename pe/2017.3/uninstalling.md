# Uninstalling

Puppet Enterprise includes a script for uninstalling. You can uninstall component infrastructure nodes or from agent nodes.

## Uninstall component nodes

The `puppet-enterprise-uninstaller` script is installed on the master, and in a split install, on the PuppetDB and console nodes. In order to uninstall, you must run the uninstaller on each component node.

### About this task

By default, the uninstaller removes the software, users, logs, cron jobs, and caches, but it leaves your modules, manifests, certificates, databases, and configuration files in place, as well as the home directories of any users it removes.

### Procedure

1.  From the component node that you want to uninstall, from the command line as root, navigate to the installer directory and run the uninstall command: `$ sudo ./puppet-enterprise-uninstaller`

    **Note:** If you don't have access to the installer directory, you can run `/opt/puppetlabs/bin/puppet-enterprise-uninstaller`.

2.  Follow prompts to uninstall.

3.  \(Optional\) If you don't uninstall the master, and you plan to reinstall on a component node at a later date, remove the agent certificate for that component from the master. On the master: `puppet cert clean <PE COMPONENT CERT NAME>`.


## Uninstall agents

You can remove the agent from nodes that you no longer want to manage.

### About this task

**Note:** Uninstalling the agent doesn't remove the node from your environment. To completely remove all traces of a node, you must also purge the node.

**Related topics**  


[Remove nodes](adding_and_removing_nodes.md#)

### Uninstall \*nix agents

The \*nix agent package doesn't include the uninstaller, so you must take additional steps to uninstall from agent nodes.

#### Procedure

1.  On the master, navigate to `/opt/puppetlabs/bin/` and copy `puppet-enterprise-uninstaller` to the agent node you want to uninstall. 

2.  On the agent node, run the installer: `sudo ./puppet-enterprise-uninstaller`

3.  Follow prompts to uninstall.

4.  \(Optional\) If you plan to reinstall on the node at a later date, remove the agent certificate for the agent from the master: `puppet cert clean <AGENT CERT NAME>`


### Uninstall Windows agents

To uninstall from a Windows node, use the Windows **Add or Remove Programs** interface, or uninstall from the command line.

#### About this task

Uninstalling removes the Puppet program directory, the agent service, and all related registry keys. The data directory remains intact, including all SSL keys. To completely remove Puppet from the system, manually delete the data directory.

#### Procedure

1.  Use the Windows **Add or Remove Programs** interface to remove the agent.

    Alternatively, you can uninstall from the command line if you have the original .msi file or know the product code of the installed MSI, for example: `msiexec /qn /norestart /x [puppet.msi|<PRODUCT_CODE>]`


### Uninstall macOS agents

The macOS uninstaller removes all aspects of the agent from that node.

#### About this task

#### Procedure

1.  On the agent node, using the macOS agent package, run the uninstaller .pkg.

2.  Follow prompts to uninstall.

3.  \(Optional\) If you plan to reinstall on the node at a later date, remove the agent certificate for the agent from the master: `puppet cert clean <AGENT CERT NAME>`


## Uninstaller options

You can use the following command-line flags to change the uninstaller's behavior.

-   `-p` — Purge additional files. With this flag, the uninstaller also removes all configuration files, modules, manifests, certificates, the home directories of any users created by the installer, and the Puppet public GPG key used for package verification.
-   `-d` — Also remove any databases created during installation.
-   `-h` — Display a help message.
-   `-n` — Run in `noop` mode; show commands that would have been run during uninstallation without actually running them.
-   `-y` — Don't ask to confirm uninstallation, assuming an answer of yes.

To remove every trace of PE from a system, run:

```
$ sudo ./puppet-enterprise-uninstaller -d -p
```

