---
layout: default
title: "Installing with an answer file"
canonical: "/pe/latest/install_automated.html"
---

You can run the Puppet Enterprise installer while logged into the target server in an automated mode that requires minimal user interaction. The installer will read pre-selected answers to the install configuration questions from an "answer file." There are two steps to the process:

1. [Obtain an answer file](#obtaining-an-answer-file).
2. Run the installer with the `-a` or `-A` flag pointed at the answer file.

The flag will cause the installer to read your choices from the answer file and act on them immediately instead of interviewing a user to customize the installation.

However, please note that an answer file installation requires you to run the installer with an answer file on each node on which you are installing a PE component. In other words, a monolithic installation will require you to run the installer with an answer file on one node, but a split installation will require you to run the installer with an answer file on three nodes.

>**Warning**: If you're performing a split installation of PE using an answer file, install the components in the following order:
>
> 1. Puppet master
> 2. Puppet DB and database support (which includes the console database)
> 3. The PE console

## Obtaining an answer file

Answer files are simply shell scripts that declare variables used by the installer, such as:

    q_install=y
    q_puppet_enterpriseconsole_install=n
    q_puppet_symlinks_install=y
    q_puppetagent_certname=webmirror1.example.com
    q_puppetagent_install=y
    q_puppetagent_server=puppet
    q_puppetmaster_install=n
    q_vendor_packages_install=y

(A full list of these variables is available in the [answer file syntax](#answer-file-syntax).)

To obtain an answer file, you can:

* Use/edit one of the example files provided in the installer's `answers` directory.
* Use the web-based installer to create an answer file. The answer file can be found at `/opt/puppetlabs/puppet/share/installer/answers`. (Note that these answers are overwritten each time you run the installer.
* Retrieve the `answers.lastrun.<HOSTNAME>` file from a node on which you've already installed PE. This file will be in the same directory from which you ran the installer.

**You must hand edit any pre-made answer file before using it**, as new nodes will need, at a minimum, a unique agent certname.

## Answer file syntax

Answer files consist of normal shell script variable assignments:

    q_database_port=3306

Boolean answers should use Y or N (case-insensitive) rather than true, false, 1, or 0.

A variable can be omitted if a prior answer ensures that it won't be used (i.e. `q_puppetmaster_certname` can be left blank if `q_puppetmaster_install` = n).

Answer files can include arbitrary bash control logic and can assign variables with commands in subshells (`$(command)`). For example, to set an agent node's certname to its fqdn:

    q_puppetagent_certname=$(hostname -f)

To set it to a UUID:

    q_puppetagent_certname=$(uuidgen)

## Editing answer files

Although you can use literal strings in an answer file for one-off installations, you should fill certain variables dynamically with bash subshells if you want your answer files to be reusable.

To run a subshell that will return the output of its command, use either the `$()` notation:

    q_puppetagent_certname=$(hostname -f)

or backticks:

    q_puppetagent_certname=`uuidgen`

Answer files can also contain arbitrary shell code and control logic, but you will probably be able to get by with a few simple name-discovery commands.

See the [complete answer file reference][answerfile] for a complete list of variables and the conditions where they're needed, or simply start editing one of the example files in `answers/`.

[answerfile]: ./install_complete_answer_file_reference.html

## Running the installer with an answer file

When you have your answer file, simply run the installer with the `-a` or `-A` option, providing your answer file as an argument:

    $ sudo ./puppet-enterprise-installer -a ~/my_answers.txt

* Installing with the `-a` option will fail if any required question variables are not set.
* Installing with the `-A` option will prompt the user for any missing answers to question variables.

**Note**: For security reasons, when the installer runs, the answer file gets permissions changes. The permissions get set to owner `root`, permissions `600`.

### After installation---answer file retrieval

When you run the installer with an answer file, the full answer file of what was actually installed can be found in the directory from where you ran the installer, at `answers.lastrun.<HOSTNAME>`.


### Answer file options

You can add the following additional flags when you run the installer:

- `-a <PATH to ANSWER FILE>`: The installer reads answers from the answer file and quits with error if an answer is missing.
- `-A <PATH to ANSWER FILE>`: The installer reads answers from the answer file and prompts for input if an answer is missing.
- `-D`: The installer will display debugging information. This option must be run with the `-a` or  `-A` flags.
- `-h`: The installer will display help information.
- `-l <PATH to LOG FILE>`: The installer will log commands and their results the installer log file. This option must be run with the `-a` or  `-A` flags.
- `-n`: The installer will run in 'no-op' mode; the installer's output will show commands that would have been run during installation without running them. This option must be run with the `-a` or `-A` flags.
- `-q`: The installer will run in quiet mode; the installation process wil not be displayed. This option must be run with the `-a` or  `-A` flags.
- `-V`: The installer will display very verbose debugging information. This option must be run with the `-a` or  `-A` flags.

> **Note**: The `-D`, `q`, `-l`, `-n`, and `-V` flags will fail with errors if they are not used in conjunction with `-a` or `-A` flags.
>
> In addition the `-q` flag will fail with an error if any errors occur during installation.

## Sample answer files

PE includes a collection of sample answer files in the `answers` directory of your distribution tarball. Sample answer file references are available for monolithic (all-in-one) and split installations. For split installations, the answer file references are broken out across the various components you will install; there is an answer file for the Puppet master, the console, and the PuppetDB components.

In addition to the answer file references for monolithic and split installations, we've included a reference for all possible answers that you can include in an answer file.

Choose from the following:

* [Complete answer file reference](./install_complete_answer_file_reference.html)
* [Monolithic (all-in-one) installation](./install_mono_answers.html)
* [Split installation (puppet master node)](./install_split_master_answers.html)
* [Split installation (console node)](./install_split_console_answers.html)
* [Split installation (PuppetDB node)](./install_split_puppetdb_answers.html)
