---
layout: default
title: "Puppet's Commands"
---

[facter_cli]: /facter/latest
[hiera_cli]: /hiera/latest/command_line.markdown
[arch]: ./architecture.markdown
[agent_unix]: ./services_agent_unix.markdown
[agent_windows]: ./services_agent_windows.markdown
[agent_man]: ./man/agent.markdown
[master_rack]: ./services_master_rack.markdown
[master_webrick]: ./services_master_webrick.markdown
[master_man]: ./man/master.markdown
[apply]: ./services_apply.markdown
[apply_man]: ./man/apply.markdown
[cert_man]: ./man/cert.markdown
[puppet forge]: https://forge.puppetlabs.com
[module fundamentals]: ./modules_fundamentals.markdown
[installing modules]: ./modules_installing.markdown
[publishing modules on the puppet forge]: ./modules_publishing.markdown
[module_man]: ./man/module.markdown
[resource_man]: ./man/resource.markdown
[about puppet's settings]: ./config_about_settings.markdown
[checking values of settings]: ./config_print.markdown
[editing settings on the command line]: ./config_set.markdown
[short list of important settings]: ./config_important_settings.markdown
[config_man]: ./man/config.markdown
[parser_man]: ./man/parser.markdown
[help_man]: ./man/help.markdown
[man_man]: ./man/man.markdown
[all_manpages]: ./man/
[about_server]: /puppetserver/1.0/services_master_puppetserver.markdown
[server_vs_passenger]: /puppetserver/1.0/puppetserver_vs_passenger.markdown
[subcommands]: /puppetserver/1.0/subcommands.markdown


Puppet's command line interface consists of a single `puppet` command with many subcommands.

[Puppet Server][about_server] and Puppet's companion utilities, [Facter][facter_cli] and [Hiera][hiera_cli], have their own command line interfaces, which differ slightly from Puppet's.


Core Services
-----

The following subcommands are the main applications Puppet uses to manage systems. Every user should understand what they do.

### Puppet Agent

Puppet agent manages systems, with the help of a Puppet master. It requests a configuration catalog from a Puppet master server, then ensures that all resources in that catalog are in their desired state.

For more information, see:

* [Overview of Puppet's Architecture][arch]
* [Puppet Agent on \*nix Systems][agent_unix]
* [Puppet Agent on Windows Systems][agent_windows]
* [Puppet Agent's Man Page][agent_man]

### Puppet Master

Puppet master compiles and serves configuration catalogs for any number of Puppet agents, using Puppet modules and various other data sources.

For more information, see:

* [Overview of Puppet's Architecture][arch]
* [The Rack Puppet Master][master_rack]
* [The WEBrick Puppet Master][master_webrick]
* [Puppet Master's Man Page][master_man]

### Puppet Server

Puppet Server compiles configurations for any number of Puppet agents, using Puppet code and various other data sources. It provides the same services as the classic Puppet master application and is meant to replace an Apache/Passenger Puppet master stack. 

Puppet Server has its own subcommand, `puppetserver`, which isn't prefaced by the usual `puppet` subcommand.

For more information, see:

* [Overview of Puppet's Architecture][arch]
* [Puppet Server][about_server]
* [Puppet Server vs. Apache/Passenger master][server_vs_passenger]
* [Puppet Server Subcommands][subcommands]

### Puppet Apply

Puppet apply manages systems without needing to contact a Puppet master server. It compiles its own configuration catalog, using Puppet modules and various other data sources, then immediately applies the catalog.

For more information, see:

* [Overview of Puppet's architecture][arch]
* [Puppet Apply][apply]
* [Puppet Apply's Man Page][apply_man]


Administrative Tools
-----

### Puppet Cert

Puppet cert helps manage Puppet's built-in certificate authority (CA). It runs on the same server as the Puppet master application. You can use it to sign and revoke agent certificates.

For more information, see:

* [Puppet Cert's Man Page][cert_man]

### Puppet Module

Puppet module is a multi-purpose tool for working with Puppet modules. It can install and upgrade new modules from the [Puppet Forge][], help generate new modules, and package modules for public release.

For more information, see:

* [Module Fundamentals][]
* [Installing Modules][]
* [Publishing Modules on the Puppet Forge][]
* [Puppet Module's Man Page][module_man]

### Puppet Resource

Puppet resource lets you interactively inspect and manipulate resources on a system. It can work with any resource type Puppet knows about.

For more information, see:

* [Learning Puppet: Resources and the RAL](/learning/ral.html)
* [Puppet Resource's Man Page][resource_man]

### Puppet Config

Puppet config lets you view and change Puppet's settings.

For more information, see:

* [About Puppet's Settings][]
* [Checking Values of Settings][]
* [Editing Settings on the Command Line][]
* [Short List of Important Settings][]
* [Puppet Config's Man Page][config_man]


Accessories
-----

### Puppet Parser

Puppet parser lets you validate Puppet code to make sure it contains no syntax errors. It can be a useful part of your continuous integration toolchain.

For more information, see:

* [Puppet Parser's Man Page][parser_man]

### Puppet Help and Puppet Man

Puppet help and Puppet man can display online help for Puppet's other subcommands.

For more information, see:

* [Puppet Help's Man Page][help_man]
* [Puppet Man's Man Page][man_man]


Full List of Subcommands
-----

Puppet has other subcommands, most of which aren't as generally useful as the ones listed above. For a complete list, see:

* [List of Puppet's Subcommands][all_manpages]
