---
layout: default
title: "Split install console answer file reference"
canonical: "/pe/latest/install_split_console_answers.html"
---

The following example answers can be used to perform an automated split installation of PE on the node assigned to the console component.

A .txt file version can be found in the `answers` directory of the PE installer tarball.

See our guide to [installing with an answer file](./install_automated.html) for more details. In addition, [the complete answer file reference](./install_complete_answer_file_reference.html) contains all answers that you can use to install PE.

>**Warning**: If you're performing a split installation of PE using an answer file, install the components in the following order:
>
> 1. Puppet master
> 2. Puppet DB and database support (which includes the console database)
> 3. The PE console

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

#### `q_use_application_services`

**Y or N** --- Whether or not to enable Application Orchestration. If you choose not to enable this during installation, you can enable it at a later time.

### Component answers

These answers are always needed.

#### `q_puppetmaster_install=n`

**Y** or **N** --- Whether to install the Puppet master component.

#### `q_puppet_enterpriseconsole_install=y`

**Y** or **N** --- Whether to install the console component.

#### `q_puppetmaster_r10k_remote='git@<your.git.server.com>:puppet/control.git'`

**String** --- The git url to be passed to the `r10k.yaml` file. This can be any URL that is supported by r10k (and normally git). This answer is OPTIONAL and only needed if you would like r10k configured when Puppet Enterprise is installed.

#### `q_puppetmaster_r10k_private_key=/root/.ssh/id_dsa`

**String** --- The local filesystem path on the Puppet master where the SSH private key can be found and used by r10k. This answer is OPTIONAL and only needed if you would like r10k configured when Puppet Enterprise is installed. This must be specified in conjunction with `q_puppetmaster_r10k_remote`.

#### `q_puppetmaster_code_manager_auto_configure`

**Y** or **No** --- (Install only) Whether or not to automatically configure the code manager service.

### Additional component answers

These answers are optional.

#### `q_puppetagent_install`

**Y** or **N** --- Whether to install the Puppet agent component.

### Puppet agent answers

These answers are always needed.

#### `q_puppetagent_certname=pe-console.<your local domain>`

**String** --- An identifying string for this agent node. This per-node ID must be unique across your entire site. Fully qualified domain names are often used as agent certnames.

#### `q_puppetagent_server=pe-master.<your local domain>`

**String** --- The hostname of the Puppet master server. For the agent to trust the master’s certificate, this must be one of the valid DNS names you chose when installing the Puppet master.

#### `q_fail_on_unsuccessful_master_lookup=y`

**Y** or **N** --- Whether to quit the install if the Puppet master cannot be reached.

#### `q_skip_master_verification=n`

**Y** or **N** --- This is a silent install option, default is N. When set to "Y", the installer will skip master verification which allows the user to deploy agents when they know the master won’t be available.

### Puppet master answers

These answers are generally needed if you are installing the Puppet master component.

#### `q_puppet_enterpriseconsole_auth_password=<your password>`

**String** --- The password for the console’s admin user. Must be longer than eight characters.

#### `q_public_hostname=`

**String** --- A publicly accessible hostname where the console can be accessed if the host name resolves to a private interface (e.g., Amazon EC2). This is set automatically by the installer on EC2 nodes, but can be set manually in environments with multiple hostnames.

#### `q_puppetmaster_r10k_remote='git@<your.git.server.com>:puppet/control.git'`

**String** --- The git url to be passed to the `r10k.yaml` file. This can be any URL that is supported by r10k (and normally git). This answer is OPTIONAL and only needed if you would like r10k configured when Puppet Enterprise is installed.

#### `q_puppetmaster_r10k_private_key=/root/.ssh/id_dsa`

**String** --- The local filesystem path on the Puppet master where the SSH private key can be found and used by r10k. This answer is OPTIONAL and only needed if you would like r10k configured when Puppet Enterprise is installed. This must be specified in conjunction with `q_puppetmaster_r10k_remote`.

#### `q_puppetmaster_code_manager_auto_configure`

**Y** or **No** --- (Install only) Whether or not to automatically configure the code manager service.

### Additional Puppet master answers

These answers are optional.

#### `q_tarball_server`

**String** --- The location from which PE agent tarballs will be downloaded before installation. Note that agent tarballs are only available for certain operating systems. For details, see the [PE agent installation instructions](./install_agents.html).

### Additional console answers

#### `q_puppet_enterpriseconsole_master_hostname`

**String** --- The hostname of the server running the master component. Only needed in a split install.

### Database support answers

These answers are only needed if you are installing the database support component.

#### `q_puppetdb_database_name=pe-puppetdb`

**String** --- The database PuppetDB will use.

#### `q_puppetdb_database_password=strongpassword1748`

**String** --- The password for PuppetDB’s root user.

#### `q_puppetdb_database_user=pe-puppetdb`

**String** --- PuppetDB’s root user name.

#### `q_puppetdb_hostname=pe-puppetdb.localdomain`

**String** --- The hostname of the server running PuppetDB.

#### `q_rbac_database_name=pe-rbac`

**String** --- The database the RBAC service will use.

#### `q_rbac_database_password=<your password>`

**String** --- The password for the RBAC service root user.

#### `q_rbac_database_user=pe-rbac`

**String** --- The RBAC service root user name.

#### `q_activity_database_name=pe-activity`

**String** --- The database the activity service will use.

#### `q_activity_database_password=<your password>`

**String** --- The password for the activity service root user.

#### `q_activity_database_user=pe-activity`

**String** --- The activity service root user name.

#### `q_orchestrator_database_name=pe-orchestrator`

**String** --- The database the orchestrator service will use.

#### `q_orchestrator_database_password=<your password>`

**String** --- The password for the orchestrator service root user.

#### `q_orchestrator_database_user=pe-orchestrator`

**String** --- The orchestrator service root user name.

#### `q_classifier_database_name=pe-classifier`

**String** --- The database the node classifier will use.

#### `q_classifier_database_password=<your password>`

**String** --- The password for the node classifer root user.

#### `q_classifier_database_user=pe-classifier`

**String** --- The node classifier root user name.

### Additional database support answers

#### `q_puppetdb_plaintext_port`

**Integer** --- The port on which PuppetDB accepts plain-text HTTP connections (default port is 8080).


