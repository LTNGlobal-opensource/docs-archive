---
layout: default
title: "Language: Resource Defaults"
---

[sitemanifest]: ./dirs_manifest.markdown
[dynamic_scope]: ./lang_scope.html#scope-lookup-rules
[resource]: ./lang_resources.markdown
[definedtypes]: ./lang_defined_types.markdown
[node]: ./lang_node_definitions.markdown

Resource defaults let you set default attribute values for a given resource type. Any resource declaration within the area of effect that omits those attributes will inherit the default values.

Syntax
-----

~~~ ruby
    Exec {
      path        => '/usr/bin:/bin:/usr/sbin:/sbin',
      environment => 'RUBYLIB=/opt/puppet/lib/ruby/site_ruby/1.8/',
      logoutput   => true,
      timeout     => 180,
    }
~~~

The general form of resource defaults is:

* The resource type, capitalized. (If the type has a namespace separator (`::`) in its name, every segment must be capitalized. E.g., `Concat::Fragment`.)
* An opening curly brace.
* Any number of attribute and value pairs.
* A closing curly brace.

You can specify defaults for any resource type in Puppet, including [defined types][definedtypes].

Behavior
-----

Within the [area of effect](#area-of-effect), every resource of the specified type that omits a given attribute will inherit that attribute's default value.

Attributes that are set explicitly in a resource declaration will always override any default value.

Resource defaults are **parse-order independent.** A default will affect resource declarations written both above and below it.

### Overriding Defaults From Parent Scopes

Resource defaults declared in the local scope will override any defaults received from parent scopes.

Overriding of resource defaults is **per attribute,** not per block of attributes. Thus, local and inherited resource defaults that don't conflict with each other will be merged together.

### Area of Effect

Although Puppet 3.x no longer does dynamic variable lookup, it still uses dynamic scope for resource defaults. [See here for a full description of scope rules][dynamic_scope].

You can declare global resource defaults in the [site manifest][sitemanifest] outside any [node definition][node].


