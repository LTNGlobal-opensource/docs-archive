---
layout: default
title: "Future Parser: Namespaces and Autoloading"
---

[classes]: ./future_lang_classes.markdown
[define]: ./future_lang_defined_types.markdown
[variables]: ./future_lang_variables.markdown
[modulepath]: ./modules_fundamentals.html#the-modulepath
[module]: ./modules_fundamentals.markdown
[scopes]: ./future_lang_scope.markdown
[include]: ./future_lang_classes.html#using-include
[PUP-121]: https://tickets.puppetlabs.com/browse/PUP-121
[inherits]: ./future_lang_classes.html#inheritance
[allowed]: ./future_lang_reserved.html#classes-and-defined-types
[relative_below]: #aside-historical-context


[Class][classes] and [defined type][define] names may be broken up into segments called **namespaces.** Namespaces tell the autoloader how to find the class or defined type in your [modules][module].

> **Warning:** [PUP-121](https://tickets.puppetlabs.com/browse/PUP-121) wasn't actually fixed as intended, and [relative namespacing](./lang_namespaces.html#relative-name-lookup-and-incorrect-name-resolution) remains a problem with the future parser in Puppet 3.8.1. It _is_ fixed in Puppet 4.0 and later. See [PUP-4818](https://tickets.puppetlabs.com/browse/PUP-4818) for more info.

Syntax
-----

Puppet [class][classes] and [defined type][define] names may consist of any number of namespace segments separated by the `::` (double colon) namespace separator. (This separator is analogous to the `/` \[slash\] in a file path.)

~~~ ruby
    class apache { ... }
    class apache::mod { ... }
    class apache::mod::passenger { ... }
    define apache::vhost { ... }
~~~

### Nested Definitions

If a class/defined type is defined inside another class/defined type definition, its name goes under the exterior definition's namespace.

This causes its real name to be something other than the name with which it was defined. For example:

~~~ ruby
    class first {
      class second {
        ...
      }
    }
~~~

In the above code, the interior class's real name is `first::second`, but searching your code for that real name will turn up nothing. Note also that it causes class `first::second` to be defined in the wrong file; see [Handling Missing Files](#handling-missing-files) below.

In short: please never do this, but you might encounter it in extraordinarily old Puppet code.

Autoloader Behavior
-----

When a class or defined resource is declared, Puppet will use its full name to find the class or defined type in your modules. Every class and defined type should be in its own file in the module's `manifests` directory, and each file should have the `.pp` file extension.

Names map to file locations as follows:

* The **first** segment in a name (excluding the empty "top" namespace) identifies the [module][]. (If this is the **only** segment, the file name will be `init.pp`.)
* The **last** segment identifies the file name (minus the `.pp` extension).
* Any segments **between** the first and last are subdirectories under the `manifests` directory.

Thus, every class or defined type name maps directly to a file path within Puppet's [`modulepath`][modulepath]:

name                     | file path
------------------------ | ---------
`apache`                 | `<MODULE DIRECTORY>/apache/manifests/init.pp`
`apache::mod`            | `<MODULE DIRECTORY>/apache/manifests/mod.pp`
`apache::mod::passenger` | `<MODULE DIRECTORY>/apache/manifests/mod/passenger.pp`

Note again that `init.pp` always contains a class or defined type named after the module, and any other `.pp` file contains a class or defined type with at least two namespace segments. (That is, `apache.pp` would contain a class named `apache::apache`.) This also means you can't have a class named `<MODULE NAME>::init`.

### Handling Missing Files

If the manifest file that corresponds to a name doesn't exist, Puppet will continue to look for the requested class or defined type. It does this by removing the final segment of the name and trying to load the corresponding file, continuing to fall back until it reaches the module's `init.pp` file.

Puppet will load the first file it finds like this, and will raise an error if that file doesn't contain the requested class or defined type.

This behavior allows you to put a class or defined type in the wrong file and still have it work, but please don't do that.
