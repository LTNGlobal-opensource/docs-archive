# Internationalization known issues

These are the known issues for UTF-8 support in this release.

## Console password must be ASCII

Supported web browsers forbid non-ASCII characters in a login password field. In the `pe.conf` file, you can specify a non-ASCII password for the console admin password, but you can't use the password to log into the console in your browser.

## Non-ASCII characters not supported in some names in the Puppet language

When naming the following items in Puppet, you must use ASCII characters:

-   Environments
-   Variables
-   Classes
-   Resource types
-   Modules
-   Parameters
-   Tags

## Some operating systems forbid non-ASCII names for several types of resources

Some operating systems allow only ASCII characters in the `title` and `namevar` of certain resource types. For example, the user and group resources on RHEL and CentOS may contain only ASCII characters in `title` and `namevar`.

## Non-ASCII file names can cause errors when used in concat fragments

When using concat fragments, an error occurs if a `source` attribute is specified and the `source` attribute points to a file name that contains non-ASCII characters.

## Non-ASCII file names can cause errors in the file resource

In a file resource, an error occurs if the `source` attribute is set to a file name containing non-ASCII characters.

## Ruby can corrupt the path fact and environment variable on Windows

There is a bug in Ruby that can corrupt the environment variable names and values. This bug causes corruption for only some codepages.

The same bug can cause the `path` fact to be cached in a corrupt state.

