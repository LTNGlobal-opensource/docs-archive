---
layout: default
title: "Monolithic Puppet Enterprise install answer file reference"
canonical: "/pe/latest/install_mono_answers.html"
---

The following example answers can be used to perform an automated monolithic (all-in-one) installation of PE.

A .txt file version can be found in the `answers` directory of the PE installer tarball.

See [installing with an answer file](./install_automated.html) for more details. In addition, [the complete answer file reference](./install_complete_answer_file_reference.html) contains all answers that you can use to install PE.

### Global answers

These answers are always needed.

#### `q_install=y`

**Y** or **N** --- Whether to install. Answer files must set this to "Y".

#### `q_vendor_packages_install=y`

**Y** or **N** --- Whether the installer has permission to install additional packages from the OS’s repositories. If this is set to N, the installation will fail if the installer detects missing dependencies.

### Additional global answers

These answers are optional.

#### `q_run_updtvpkg`

**Y** or **N** --- Only used on AIX. Whether to run `updtvpkg` command to add info about native libraries to the RPM database. The answer should usually be "Y", unelss you have special needs around the RPM.

### Components

These answers are always needed.

#### `q_puppetmaster_install=y`

**Y** or **N** --- Whether to install the Puppet master component.

#### `q_all_in_one_install=y`

**Y** or **N** --- Whether or not the installation is an all-in-one installation, (i.e., are PuppetDB and the console also being installed on this node).

### Additional component answers

These answers are optional.

#### `q_puppetagent_install`

**Y** or **N** --- Whether to install the Puppet agent component.

#### `q_use_application_services`

**Y or N** --- Whether or not to enable Application Orchestration. If you choose not to enable this during installation, you can enable it at a later time.

### Puppet agent answers

These answers are always needed.

#### `q_puppetagent_certname=pe-puppet.<your local domain>`

**String** --- An identifying string for this agent node. This per-node ID must be unique across your entire site. Fully qualified domain names are often used as agent certnames.

### Puppet master answers

These answers are generally needed if you are installing the Puppet master component.

#### `q_puppetmaster_certname=pe-puppet.<your local domain>`

**String** --- An identifying string for the Puppet master. This ID must be unique across your entire site. The server’s fully qualified domain name is often used as the Puppet master’s certname.

#### `q_puppetmaster_dnsaltnames=pe-puppet,pe-puppet.<your local domain>`

**String** --- Valid DNS names at which the Puppet master can be reached. Must be a comma-separated list. In a normal installation, defaults to `<hostname>,<hostname.domain>,puppet,puppet.<domain>`.

#### `q_pe_check_for_updates=y`

**y** or **n**; **MUST BE LOWERCASE** --- Whether to check for updates whenever the `pe-puppetserver` service restarts. To get the correct update info, the server will pass some basic, anonymous info to Puppet's servers. Specifically, it will transmit:

   * the IP address of the client
   * the type and version of the client’s OS
   * the installed version of PE
   * the number of nodes licensed and the number of nodes used

If you wish to disable update checks (e.g. if your company policy forbids transmitting this information), you will need to set this to "n". You can also disable checking after installation by editing the `/etc/puppetlabs/installer/answers.install` file.

#### `q_puppet_enterpriseconsole_httpd_port=443`

**Integer** --- The port on which to serve the console. The default is port 443, which will allow access to the console from a web browser without manually specifying a port. If port 443 is not available, the installer will try port 3000, 3001, 3002, 3003, 3004, and 3005.

#### `q_puppet_enterpriseconsole_auth_password=<your password>`

**String** --- The password for the console’s admin user. Must be longer than eight characters.

#### `q_public_hostname=`

**String** --- A publicly accessible hostname where the console can be accessed if the host name resolves to a private interface (e.g., Amazon EC2). This is set automatically by the installer on EC2 nodes, but can be set manually in environments with multiple hostnames.

### Additional Puppet master answers

These answers are optional.

#### `q_tarball_server`

**String** --- The location from which PE agent tarballs will be downloaded before installation. Note that agent tarballs are only available for certain operating systems. For details, see the [PE agent installation instructions](./install_agents.html).

#### `q_exit_and_update_auth_conf=y`

**Y** or **N** --- (Upgrade only) Whether to accept the presence of a modified auth.conf file when upgrading. 

#### `q_puppetmaster_r10k_remote='git@<your.git.server.com>:puppet/control.git'`

**String** --- The git url to be passed to the `r10k.yaml` file. This can be any URL that is supported by r10k (and normally git). This answer is OPTIONAL and only needed if you would like r10k configured when Puppet Enterprise is installed.

#### `q_puppetmaster_r10k_private_key=/root/.ssh/id_dsa`

**String** --- The local filesystem path on the Puppet master where the SSH private key can be found and used by r10k. This answer is OPTIONAL and only needed if you would like r10k configured when Puppet Enterprise is installed. This must be specified in conjunction with `q_puppetmaster_r10k_remote`.

#### `q_puppetmaster_code_manager_auto_configure`

**Y** or **No** --- (Install only) Whether or not to automatically configure the code manager service.

#### `q_puppetmaster_file_sync_service_enabled`

**Y** or **No** --- Whether or not to enable the file sync service.

#### `q_puppetmaster_external_node_terminus`

**Y** or **N** --- Use an external node classifier. **Important**: Use of an external node classifier is **neither supported nor tested**, and you will not be able to access classification-related features in the console, such as Node Management or RBAC.


### Database support answers

These answers are only needed if you are installing the database support component.

#### `q_puppetdb_database_name=pe-puppetdb`

**String** --- The database PuppetDB will use.

#### `q_puppetdb_database_password=<your password>`

**String** --- The password for PuppetDB’s root user.

#### `q_puppetdb_database_user=pe-puppetdb`

**String** — PuppetDB’s root user name.

### Additional database support answers

#### `q_puppetdb_plaintext_port`

**Integer** --- The port on which PuppetDB accepts plain-text HTTP connections (default port is 8080).



