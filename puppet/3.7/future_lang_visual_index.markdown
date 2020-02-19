---
layout: default
title: "Future Parser: Visual Index"
---


[resource]: ./future_lang_resources.markdown
[type]: ./future_lang_resources.html#resource-types
[title]: ./future_lang_resources.html#title
[attribute]: ./future_lang_resources.html#attributes
[string]: ./future_lang_data_string.markdown
[function]: ./future_lang_functions.markdown
[rvalue]: ./future_lang_functions.html#behavior
[template_func]: /guides/templating.markdown
[module]: modules_fundamentals.markdown
[argument]: ./future_lang_functions.html#arguments
[relationship_meta]: ./future_lang_relationships.html#relationship-metaparameters
[refs]: ./future_lang_data_resource_reference.markdown
[chaining]: ./future_lang_relationships.html#chaining-arrows
[variable]: ./future_lang_variables.markdown
[array]: ./future_lang_data_array.markdown
[hash]: ./future_lang_data_hash.markdown
[interpolation]: ./future_lang_data_string.html#variable-interpolation
[class_def]: ./future_lang_classes.html#defining-classes
[class_decl]: ./future_lang_classes.html#declaring-classes
[defined_type]: ./future_lang_defined_types.markdown
[namespace]: ./future_lang_namespaces.markdown
[defined_resource]: ./future_lang_defined_types.html#declaring-an-instance
[node]: ./future_lang_node_definitions.markdown
[regex_node]: ./future_lang_node_definitions.html#regular-expression-names
[comments]: ./future_lang_comments.markdown
[if]: ./future_lang_conditional.html#if-statements
[expressions]: ./future_lang_expressions.markdown
[built_in]: ./future_lang_variables.html#facts-and-built-in-variables
[facts]: ./future_lang_variables.html#facts
[regex]: ./future_lang_data_regexp.markdown
[regex_match]: ./future_lang_expressions.html#regex-match
[in]: ./future_lang_expressions.html#in
[case]: ./future_lang_conditional.html#case-statements
[selector]: ./future_lang_conditional.html#selectors
[collector]: ./future_lang_collectors.markdown
[export_collector]: ./future_lang_collectors.html#exported-resource-collectors
[export]: ./future_lang_exported.markdown
[defaults]: ./future_lang_defaults.markdown
[override]: ./future_lang_classes.html#overriding-resource-attributes
[inherits]: ./future_lang_classes.html#inheritance
[coll_override]: ./future_lang_resources_advanced.html#amending-attributes-with-a-collector
[virtual]: ./future_lang_virtual.markdown

This page can help you find syntax elements when you can't remember their names.


~~~ ruby
    file {'ntp.conf':
      path    => '/etc/ntp.conf',
      ensure  => file,
      content => template('ntp/ntp.conf'),
      owner   => 'root',
      mode    => '0644',
    }
~~~

↑ A [resource declaration][resource].

* `file`: The [resource type][type]
* `ntp.conf`: The [title][]
* `path`: An [attribute][]
* `'/etc/ntp.conf'`: The value of an [attribute][]; in this case, a [string][]
* `template('ntp/ntp.conf')`: A [function][] call that [returns a value][rvalue]; in this case, the [`template`][template_func] function, with the name of a template in a [module][] as its [argument][]

~~~ ruby
    package {'ntp':
      ensure => installed,
      before => File['ntp.conf'],
    }
    service {'ntpd':
      ensure    => running,
      subscribe => File['ntp.conf'],
    }
~~~

↑ Two resources using the `before` and `subscribe` [relationship metaparameters][relationship_meta] (which accept [resource references][refs]).

~~~ ruby
    Package['ntp'] -> File['ntp.conf'] ~> Service['ntpd']
~~~

↑ [Chaining arrows][chaining] forming relationships between three resources, using [resource references][refs].

~~~ ruby
    $package_list = ['ntp', 'apache2', 'vim-nox', 'wget']
~~~

↑ A [variable][] being assigned an [array][] value.

~~~ ruby
    $myhash = { key => { subkey => 'b' }}
~~~

↑ A [variable][] being assigned a [hash][] value.

~~~ ruby
    ...
    content => "Managed by puppet master version ${serverversion}"
~~~

↑ A master-provided [built-in variable][built_in] being [interpolated into a double-quoted string][interpolation] (with optional curly braces).


~~~ ruby
    class ntp {
      package {'ntp':
        ...
      }
      ...
    }
~~~

↑ A [class definition][class_def], which makes a class avaliable for later use.

~~~ ruby
    include ntp
    require ntp
    class {'ntp':}
~~~

↑ [Declaring a class][class_decl] in three different ways: with the `include` function, with the `require` function, and with the resource-like syntax. Declaring a class causes the resources in it to be managed.


~~~ ruby
    define apache::vhost ($port, $docroot, $servername = $title, $vhost_name = '*') {
      include apache
      include apache::params
      $vhost_dir = $apache::params::vhost_dir
      file { "${vhost_dir}/${servername}.conf":
          content => template('apache/vhost-default.conf.erb'),
          owner   => 'www',
          group   => 'www',
          mode    => '644',
          require => Package['httpd'],
          notify  => Service['httpd'],
      }
    }
~~~

↑ A [defined type][defined_type], which makes a new resource type available. Note that the name of that resource type has two [namespace segments][namespace].

~~~ ruby
    apache::vhost {'homepages':
      port    => 8081,
      docroot => '/var/www-testhost',
    }
~~~

↑ [Declaring an instance][defined_resource] of the resource type defined above.

~~~ ruby
    Apache::Vhost['homepages']
~~~

↑ A [resource reference][refs] to the defined resource declared above. Note that every [namespace segment][namespace] must be capitalized.

~~~ ruby
    node 'www1.example.com' {
      include common
      include apache
      include squid
    }
~~~

↑ A [node definition][node].

~~~ ruby
    node /^www\d+$/ {
      include common
    }
~~~

↑ A [regular expression node definition][regex_node].

~~~ ruby
    # comment
    /* comment */
~~~

↑ Two [comments][].


~~~ ruby
    if str2bool("$is_virtual") {
      warning( 'Tried to include class ntp on virtual machine; this node may be misclassified.' )
    }
    elsif $operatingsystem == 'Darwin' {
      warning( 'This NTP module does not yet work on our Mac laptops.' )
    else {
      include ntp
    }
~~~

↑ An [if statement][if], whose conditions are [expressions][] that use agent-provided [facts][].


~~~ ruby
    if $hostname =~ /^www(\d+)\./ {
      notify { "Welcome web server #$1": }
    }
~~~

↑ An [if statement][if] using a [regular expression][regex] and the [regex match operator][regex_match].

~~~ ruby
    if 'www' in $hostname {
      ...
    }
~~~

↑ An [if statement][if] using an [`in` expression][in]

~~~ ruby
    case $operatingsystem {
      'Solaris':          { include role::solaris }
      'RedHat', 'CentOS': { include role::redhat  }
      /^(Debian|Ubuntu)$/:{ include role::debian  }
      default:            { include role::generic }
    }
~~~

↑ A [case statement][case].

~~~ ruby
    $rootgroup = $osfamily ? {
        'Solaris'          => 'wheel',
        /(Darwin|FreeBSD)/ => 'wheel',
        default            => 'root',
    }
~~~

↑ A [selector statement][selector] being used to set the value of the `$rootgroup` [variable][].

~~~ ruby
    User <| groups == 'admin' |>
~~~

↑ A [resource collector][collector], sometimes called the "spaceship operator."

~~~ ruby
    Concat::Fragment <<| tag == "bacula-storage-dir-${bacula_director}" |>>
~~~

↑ An [exported resource collector][export_collector], which works with [exported resources][export]

~~~ ruby
    Exec {
      path        => '/usr/bin:/bin:/usr/sbin:/sbin',
      environment => 'RUBYLIB=/opt/puppet/lib/ruby/site_ruby/1.8/',
      logoutput   => true,
      timeout     => 180,
    }
~~~

↑ A [resource default][defaults] for the `exec` resource type.

~~~ ruby
    Exec['update_migrations'] {
      environment => 'RUBYLIB=/usr/lib/ruby/site_ruby/1.8/',
    }
~~~

↑ A [resource override][override], which will only work in an [inherited class][inherits].

~~~ ruby
    Exec <| title == 'update_migrations' |> {
      environment => 'RUBYLIB=/usr/lib/ruby/site_ruby/1.8/',
    }
~~~

↑ A [resource override using a collector][coll_override], which will work anywhere. Dangerous, but very useful in rare cases.


~~~ ruby
    @user {'deploy':
      uid     => 2004,
      comment => 'Deployment User',
      group   => www-data,
      groups  => ["enterprise"],
      tag     => [deploy, web],
    }
~~~

↑ A [virtual resource][virtual].


~~~ ruby
    @@nagios_service { "check_zfs${hostname}":
      use                 => 'generic-service',
      host_name           => "$fqdn",
      check_command       => 'check_nrpe_1arg!check_zfs',
      service_description => "check_zfs${hostname}",
      target              => '/etc/nagios3/conf.d/nagios_service.cfg',
      notify              => Service[$nagios::params::nagios_service],
    }
~~~

↑ An [exported resource][export] declaration.

