---
layout: default
title: "Installing Mac OS X agents"
canonical: "/pe/latest/install_osx.html"
---

>**Note**: Mac OS X is an agent only platform. Mavericks (10.9), Yosemite (10.10), or El Capitan (10.11) are required.

Mac OS X agents provide consistent automated management of Apple laptops and desktops from the same Puppet Enterprise infrastructure that manages your servers. Capabilities include PE core functionality, plus OS X-specific capabilities:

* Package installation via DMG and PKG
* Service management via LaunchD
* Directory Services integration for local user/group management
* Inventory facts via System Profiler

> **Warning**: In PE agent certnames need to be lowercase. For Mac OS X agents, the certname is derived from the name of the machine (e.g., My-Example-Mac>. To prevent installation issues, you will want to make sure the name of your machine uses lowercases letters. You can make this change in **System Preferences** > **Sharing** > **Computer Name** > **Edit**.
>
> To make this change from the command line, run the following commands:
>
>  1. `sudo scutil --set ComputerName <newname>`
>  2. `sudo scutil --set LocalHostName <newname>`
>  3. `sudo scutil --set HostName <newname>`
>
>If you don't want to change your computer's name, you can also enter the agent certname in all lowercase letters when prompted by the installer.

### Install with Puppet Enterprise package management

To install the agent on a node running Mac OS X using PE package management tools, refer to [Installing Agents Using PE Package Management](./install_agents.html#installing-puppet-agents-with-the-package-installation-script).

### Install from Finder

To install the agent on a node running Mac OS X using Finder:

1. Download the [OS X PE agent package](http://puppetlabs.com/download-puppet-enterprise).
2. Open the PE .dmg and click the installer .pkg.
3. Follow the instructions in the installer dialog. You will need to include the Puppet master's hostname and the agent's certname.

   The installer automatically generates a certificate and contacts the master to request that the certificate be signed.

### Install from command line

To install the agent on a node running Mac OS X using the command line:

1. SSH into your OS X node as root or a sudo user. Note that you will be in `/var/root`.
2. Download the [OS X PE agent package](http://puppetlabs.com/download-puppet-enterprise).
3. Run `sudo hdiutil mount <DMGFILE>`.

   You will see a line that ends, `/Volumes/puppet-agent-VERSION`. This is the mount point for the virtual volume created from the disk image.

4. Run `cd /Volumes/puppet-agent-VERSION`.
5. Run `sudo installer -pkg puppet-agent-installer.pkg -target /`.
6. To verify the install, run `/opt/puppetlabs/bin/puppet --version`.

   **Tip**: Run `PATH=/opt/puppetlabs/bin:$PATH;export PATH` to add the PE binaries to your path.

7. Using the instructions in [configuring agents](./install_agents.html#configuring-agents), point the OS X agent at the correct Puppet master and set the agent's certname.
8. Kick off a Puppet run using `sudo puppet agent -t`. This will create a certificate signing request that you will need to [sign](./install_agents.html#signing-agent-certificates).

## A note about binaries and symlinks

Refer to [Puppet Enterprise binaries and symlinks](./install_basic.html#puppet-enterprise-binaries-and-symlinks) for information about the binaries and symlinks PE sets. You can also find instructions on disabling the symlinks.
