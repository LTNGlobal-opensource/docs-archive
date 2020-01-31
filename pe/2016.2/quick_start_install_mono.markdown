---
layout: default
title: "Install Puppet Enterprise quick start guide"
canonical: "/pe/latest/quick_start_install_mono.html"
---


## Overview

To get you started using Puppet Enterprise (PE) relatively quickly and efficiently, this guide walks you through the steps of a monolithic install. A monolithic deployment is best suited for users who want to evaluate PE, or for users managing up to 500 Puppet agent nodes. For larger installations, perform a [split install](./install_pe_split.html).

A monolithic PE deployment entails installing the Puppet master, the PE console, and PuppetDB all on one node. Puppet agents are installed later as part of the [agent installation quick start guide for *nix](./quick_start_install_agents_nix.html) or the [agent installation quick start guide for Windows](./quick_start_install_agents_windows.html).

Note that the Puppet master can only be installed on a *nix machine. If you plan to work from a Windows machine, see the [Windows PE installation page]({{pe}}/windows_installing.html).

For more information about the components that make up your PE deployment, visit the [installation overview](./install_basic.html) in the PE docs.

Note: This guide assumes that you’ll install a monolithic PE deployment as `root`.

## Installing the monolithic Puppet Enterprise deployment

1. Review the following general prerequisites:

>### General prerequisites and notes
>
>- See the [system requirements](./install_system_requirements.html#monolithic-installation) to ensure your hardware needs are met.
>
>- Make sure that DNS is properly configured on the machines you're installing PE on. All nodes must **know their own hostnames.** This can be done by properly configuring reverse DNS on your local DNS server, or by setting the hostname explicitly. Setting the hostname usually involves the `hostname` command and one or more configuration files, while the exact method varies by platform. In addition, all nodes must be able to **reach each other by name.** This can be done with a local DNS server, or by editing the `/etc/hosts` file on each node to point to the proper IP addresses.
>
>- Ensure that port 3000 is reachable, because the web-based installer uses this port. You can close this port when the installation is complete.

2. [Download and verify the appropriate PE tarball](./install_basic.html#downloading-puppet-enterprise), and, if needed, copy the tarball to the machine on which you'll be installing PE.

   > **Tip**: Be sure to download the full PE tarball, not the agent-only tarball. The agent-only tarball is used for [package management-based agent installation](./install_agents.html), which is not covered by this guide.

3. Unpack the tarball by running `tar -xf <tarball>`.
4. From the PE installer directory, run `sudo ./puppet-enterprise-installer`.
5. When prompted, choose the “Guided” installation option. 

   At this point, the PE installer starts a web server and provides a web address: `https://<install platform hostname>:3000`. Ensure that port 3000 is reachable. If necessary, you can close port 3000 when the installation is complete. Also, be sure to use `https`.

   >**Warning**: Leave your terminal connection open until the installation is complete; otherwise the installation will fail.

6. Copy the address into your browser.
7. When prompted, accept the security request in your browser.

   The web-based installation uses a default SSL certificate, so you must add a security exception in order to access the web-based installer. This is safe to do.

   You'll be taken to the installer start page.

8. On the start page, click **Let's get started!**.
9. Next, you'll be asked to choose your deployment type. Select **Monolithic**.
10. Choose to install the Puppet master component on the server you're running the installer from.
11. Provide the following information about the Puppet master server:

    a. **Puppet master FQDN**: The fully qualified domain name of the server you're installing PE on. For example, `master.example.com`.

    b. **DNS aliases**: A comma-separated list of aliases agent nodes can use to reach to the master. For example `master`.

12. When prompted about database support, choose the default option **Install PostgreSQL for me**.

13. Provide the following information about the PE console administrator user:

    **Console superuser password**: Create a password for the console login. The password must be at least eight characters.

    **Note**: The user name for the console administrator user is __admin__.

14. Click **Submit**.
15. On the confirm plan page, review the information you provided.

    If you need to make any changes, click **Go back** and make whatever changes are required. Otherwise, click **Continue**.

16. On the validation page, the installer verifies various configuration elements (for example, if SSH credentials are correct, if there is enough disk space, and if the OS is the same for the various components). If there are no outstanding issues, click **Deploy now**.

The installer then installs and configures Puppet Enterprise. It may also need to install additional packages from your OS's repository. **This process may take up to 15 minutes.** When the installation is complete, the installer script that was running in the terminal closes itself.

> You have now installed the Puppet master node. The Puppet master node is also an agent, and can configure itself the same way it configures the other nodes in a deployment. Stay logged in as root for further exercises.

### Log into the console

To log into the console, select the **Start using Puppet Enterprise** button that appears at the end of the web-based installer or open a web browser and point it to the address supplied by the installer, for example, `https://master.example.com`.

1. You will receive a warning about an untrusted certificate. This is because _you_ were the signing authority for the console's certificate, and your Puppet Enterprise deployment is not known to the major browser vendors as a valid signing authority. **Ignore the warning and accept the certificate**. The steps to do this [vary by browser](./console_accessing.html#accepting-the-consoles-certificate).
2. On the login page for the console, **log in** with the username **admin** and the password you provided when installing the Puppet master.

   The console UI loads in your browser.


Next: Install Agents---[Windows](./quick_start_install_agents_windows.html) or [*nix](./quick_start_install_agents_nix.html)
