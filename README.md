# puppet-logstash

A Puppet module for managing and configuring [Logstash](http://logstash.net/).

[![Build Status](https://travis-ci.org/elasticsearch/puppet-logstash.png?branch=master)](https://travis-ci.org/elasticsearch/puppet-logstash)

## Versions

This overview shows you which Puppet module and Logstash version work together.

    ------------------------------------
    | Puppet module | Logstash         |
    ------------------------------------
    | 0.0.1 - 0.1.0 | 1.1.9            |
    ------------------------------------
    | 0.2.0         | 1.1.10           |
    ------------------------------------
    | 0.3.0 - 0.3.4 | 1.1.12 - 1.1.13  |
    ------------------------------------
    | 0.4.0         | 1.2.x - 1.3.x    |
    ------------------------------------

## Important notes

### 0.4.0

Please note that this a backwards compatability breaking release.
The plugin defines have been removed im favor of config files.

If you need any help please see the [need help ?](#need-help) section.


## Requirements

* Puppet 2.7.x or better.
* The [stdlib](https://forge.puppetlabs.com/puppetlabs/stdlib) Puppet library.
* The [file_concat](https://forge.puppetlabs.com/ispavailability/file_concat) Puppet library.

## Usage Examples

The minimum viable configuration ensures that the service is running and that it will be started at boot time:

     class { 'logstash': }

Specify a particular package (version) to be installed:

     class { 'logstash':
       version => '1.3.3-1_centos'
     }

In the absense of an appropriate package for your environment it is possible to install from other sources as well.

http/https/ftp source:

     class { 'logstash':
       package_url => 'http://download.elasticsearch.org/logstash/logstash/packages/centos/logstash-1.3.3-1_centos.noarch.rpm'
     }

`puppet://` source:

     class { 'logstash':
       package_url => 'puppet:///path/to/logstash-1.3.3-1_centos.noarch.rpm'
     }

Local file source:

     class { 'logstash':
       package_url => 'file:/path/to/logstash-1.3.3-1_centos.noarch.rpm'
     }

Attempt to upgrade Logstash if a newer package is detected (`false` by default):

     class { 'logstash':
       autoupgrade => true
     }

Install everything but *disable* the service (useful for pre-configuring systems):

     class { 'logstash':
       status => 'disabled'
     }

Under normal circumstances a modification to the Logstash configuration will trigger a restart of the service. This behaviour can be disabled:

     class { 'logstash':
       restart_on_change => false
     }
     
Disable and remove Logstash entirely:

     class { 'logstash':
       ensure => 'absent'
     }     

## Configuration Overview

The Logstash configuration can be supplied as a single static file or dynamically built from multiple smaller files.

The basic usage is identical in either case: simply declare a `file` attribute as you would the [`content`](http://docs.puppetlabs.com/references/latest/type.html#file-attribute-content) attribute of the `file` type, meaning either direct content or a template.

     logstash::configfile { 'configname':
       file => template('path/to/config.file')
     }

To dynamically build a configuration, simply declare the `order` in which each section should appear - the lower the number the earlier it will appear in the resulting file (this should be a [familiar idiom](https://en.wikipedia.org/wiki/BASIC) for most).

     logstash::configfile { 'input_redis':
       file  => template('input_redis.erb'),
       order => 10
     }

     logstash::configfile { 'filter_apache':
       file  => template('filter_apache.erb'),
       order => 20
     }

     logstash::configfile { 'output_es':
       file  => template('output_es_cluster.erb')
       order => 30
     }

## Patterns

Many plugins (notably [Grok](http://logstash.net/docs/latest/filters/grok)) use *patterns*. Many are [included](https://github.com/logstash/logstash/tree/master/patterns) in Logstash already; however, additional site-specific patterns can be easily managed (you are, of course, encouraged to contribute new patterns to the community).

**N.B.** As of Logstash 1.2 the path to the additional patterns needs to be configured explicitly in the Grok configuration.

     logstash::patternfile { 'extra_patterns':
       source => 'puppet:///path/to/extra_pattern'
     }

By default the resulting filename of the pattern will match that of the source. This can be over-ridden:

     logstash::patternfile { 'extra_patterns_firewall':
       source   => 'puppet:///path/to/extra_patterns_firewall_v1',
       filename => 'extra_patterns_firewall'
     }

## Plugins

Like the patterns above, Logstash comes with a large number of [plugins](http://logstash.net/docs/latest/); however, additional site-specific plugins can be easily managed (again, you are encouraged to contribute new plugins to the community).

     logstash::plugin { 'myplugin':
       ensure => 'present',
       type   => 'input',
       source => 'puppet:///path/to/my/custom/plugin.rb'
     }

By default the resulting filename of the plugin will match that of the source. This can be over-ridden:

     logstash::plugin { 'myplugin':
       ensure   => 'present',
       type     => 'output',
       source   => 'puppet:///path/to/my/custom/plugin_v1.rb',
       filename => 'plugin.rb'
     }

## Java Install

Most sites will manage Java seperately; however, this module can attempt to install Java as well.

     class { 'logstash':
       java_install => true
     }

Specify a particular Java package (version) to be installed:

     class { 'logstash':
       java_install => true,
       java_package => 'packagename'
     }

## Service Management

Currently only the basic SysV-style [init](https://en.wikipedia.org/wiki/Init) service provider is supported but others could be implemented relatively easily (pull requests welcome).

### init

#### Defaults File

The *defaults* file (`/etc/defaults/logstash` or `/etc/sysconfig/logstash`) for the Logstash service can be populated as necessary. This can either be a static file resource or a simple key value-style  [hash](http://docs.puppetlabs.com/puppet/latest/reference/lang_datatypes.html#hashes) object, the latter being particularly well-suited to pulling out of a data source such as Hiera.

##### file source

     class { 'logstash':
       init_defaults_file => 'puppet:///path/to/defaults'
     }

##### hash representation

     $config_hash = {
       'LS_USER' => 'logstash',
       'LS_GROUP' => 'logstash',
     }

     class { 'logstash':
       init_defaults => $config_hash
     }

## Need help ?

Need help? Try #logstash on freenode irc or the logstash-users@googlegroups.com mailing list.
