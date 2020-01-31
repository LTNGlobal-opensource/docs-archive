# Installing external PostgreSQL

By default, Puppet Enterprise includes its own database backend, PE-PostgreSQL, which is installed alongside PuppetDB. If the load on your PuppetDB node is larger than it can effectively scale to \(greater than 20,000 nodes\), you can install a standalone instance of PE-PostgreSQL. 

In certain limited circumstances, you might choose to configure a PostgreSQL instance that's not managed by PE. Using unmanaged PostgreSQL increases complexity for maintenance and upgrades, so we recommend this configuration only for customers who can't use PE-PostgreSQL.

## Install standalone PE-PostgreSQL

If the load on your PuppetDB node is larger than it can effectively scale to \(greater than 20,000 nodes\), you can install a standalone instance of PE-PostgreSQL.

### Before you begin

You must have root access to the node on which you plan to install PE-PostgreSQL, as well as the ability to SSH and copy files to the node.

### About this task

### Procedure

1.  Prepare your `pe.conf` file by specifying parameters required for PostgreSQL. 

    ```
    "puppet_enterprise::puppet_master_host": "<MASTER OF MASTERS HOSTNAME>"
    "puppet_enterprise::database_host": "<PE-POSTGRESQL NODE HOSTNAME>"
    ```

2.  Follow the instructions to [Install using text install](installing_pe.md#), running the installer with the `-c` flag.

    The installer fails halfway through, because it can't contact the database. Components that rely on the database also fail to start. The next installation run corrects this issue.

3.  Copy the `pe.conf` file you created to the PE-PostgreSQL node and SSH into that node.

4.  Run the installer with the `-c` flag, using the same `pe.conf` file.

5.  When the installation process finishes on the PE-PostgreSQL node, SSH into the master of masters and complete the installation:

    ```
    puppet infrastructure configure; puppet agent -t;
    ```

    The master of masters is configured to use the standalone PE-PostgreSQL installation on the PE-PostgreSQL node.


## Install unmanaged PostgreSQL

If you use Amazon RDS, or if your business requirements dictate that databases must be managed outside of Puppet, you can configure a PostgreSQL database that's not managed by PE.

### Before you begin

You must have:

-   PostgreSQL 9.6 or later.

-   The complete certificate authority certificate chain for the external party CA, in PEM format.

-   The DNS-addressable name, username, and password for the external PostgreSQL database.


### About this task

**Important:** Using unmanaged PostgreSQL increases complexity for maintenance and upgrades, so we recommend it only for customers who can't use PE-PostgreSQL.

### Create the unmanaged PostgreSQL instance

#### Procedure

1.  Create the unmanaged PostgreSQL instance, and, if you haven't already, retrieve the necessary certificate chain and credentials from your database administrator.

    For example, for RDS, the root certificate is available [here](https://s3.amazonaws.com/rds-downloads/rds-ca-2015-root.pem).


### Create PE databases on the unmanaged PostgreSQL instance

#### Procedure

1.  Log in to the unmanaged PostgreSQL instance with the client of your choice.

2.  Create databases for the orchestrator, RBAC, activity service, and the node classifier.

    ```
    CREATE USER "pe-puppetdb" PASSWORD '<PASSWORD>';
    GRANT "pe-puppetdb" TO <ADMIN USER>;
    CREATE DATABASE "pe-puppetdb" OWNER "pe-puppetdb"
    ENCODING 'utf8' LC_CTYPE 'en_US.utf8' LC_COLLATE 'en_US.utf8' template template0;
    
    CREATE USER "pe-orchestrator" PASSWORD '<PASSWORD>';
    GRANT "pe-orchestrator" TO <ADMIN USER>;
    CREATE DATABASE "pe-orchestrator" OWNER "pe-orchestrator"
    ENCODING 'utf8' LC_CTYPE 'en_US.utf8' LC_COLLATE 'en_US.utf8' template template0;
    
    CREATE USER "pe-activity" PASSWORD '<PASSWORD>';
    GRANT "pe-activity" TO <ADMIN USER>;
    CREATE DATABASE "pe-activity" OWNER "pe-activity"
    ENCODING 'utf8' LC_CTYPE 'en_US.utf8' LC_COLLATE 'en_US.utf8' template template0;
    
    CREATE USER "pe-classifier" PASSWORD '<PASSWORD>';
    GRANT "pe-classifier" TO <ADMIN USER>;
    CREATE DATABASE "pe-classifier" OWNER "pe-classifier"
    ENCODING 'utf8' LC_CTYPE 'en_US.utf8' LC_COLLATE 'en_US.utf8' template template0;
    
    CREATE USER "pe-rbac" PASSWORD '<PASSWORD>';
    GRANT "pe-rbac" TO <ADMIN USER>;
    CREATE DATABASE "pe-rbac" OWNER "pe-rbac"
    ENCODING 'utf8' LC_CTYPE 'en_US.utf8' LC_COLLATE 'en_US.utf8' template template0;
    
    \c "pe-rbac"
    CREATE EXTENSION citext;
    CREATE EXTENSION pg_trgm;
    CREATE EXTENSION plpgsql;
    CREATE EXTENSION pgcrypto;
    
    \c "pe-orchestrator"
    CREATE EXTENSION citext;
    CREATE EXTENSION pg_trgm;
    CREATE EXTENSION plpgsql;
    
    \c "pe-puppetdb"
    CREATE EXTENSION citext;
    CREATE EXTENSION pg_trgm;
    CREATE EXTENSION plpgsql;
    CREATE EXTENSION pgcrypto;
    
    \c "pe-classifier"
    CREATE EXTENSION citext;
    CREATE EXTENSION pg_trgm;
    CREATE EXTENSION plpgsql;
    
    \c "pe-activity"
    CREATE EXTENSION citext;
    CREATE EXTENSION pg_trgm;
    CREATE EXTENSION plpgsql;
    ```


#### Results

#### What to do next

Next, install PE using [text mode](installing_pe.md#), specifying [external PostgreSQL parameters](installing_pe.md#) in `pe.conf`.

### Establish SSL between PE and the unmanaged PostgreSQL instance

#### Before you begin

Install PE using [text mode](installing_pe.md#), specifying [external PostgreSQL parameters](installing_pe.md#) in `pe.conf`.

#### Procedure

1.  Log in to the master and stop the agent service:

    ```
    /opt/puppetlabs/puppet/bin/puppet resource service puppet ensure=stopped
    ```

2.  On the master, create a location to store the CA cert from the external PostgreSQL instance, for example `/etc/puppetlabs/puppet/ssl/`.

3.  Transfer the CA cert from the unmanaged PostgreSQL instance to the directories that you created.

4.  Specify ownership and permissions for the certificate and directories.

    ```
    chown -R pe-puppet:pe-puppet <PATH TO DIRECTORY>
    chmod 664 <PATH TO DIRECTORY>/<CERT NAME>
    ```

5.  Modify your `pe.conf` file to specify database properties.

    ```
    "puppet_enterprise::profile::console::database_properties": "?ssl=true&sslfactory=org.postgresql.ssl.jdbc4.LibPQFactory&sslmode=verify-full&sslrootcert=<PATH TO EXTERNAL POSTGRESQL CA CERT>"
    "puppet_enterprise::profile::puppetdb::database_properties": "?ssl=true&sslfactory=org.postgresql.ssl.jdbc4.LibPQFactory&sslmode=verify-full&sslrootcert=<PATH TO EXTERNAL POSTGRESQL CA CERT>"
    "puppet_enterprise::profile::orchestrator::database_properties": "?ssl=true&sslfactory=org.postgresql.ssl.jdbc4.LibPQFactory&sslmode=verify-full&sslrootcert=<PATH TO EXTERNAL POSTGRESQL CA CERT>"
    "puppet_enterprise::database_ssl": true
    "puppet_enterprise::database_cert_auth": false
    ```

6.  On the master, run Puppet.

7.  On the master, start the agent service: `puppet resource service puppet ensure=running`


