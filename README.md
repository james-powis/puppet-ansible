# Ansible puppet module

When puppet and ansible work together for better orchestration

* [Definitions](#definitions)
* [Description](#description)
* [Requirements](#requirements)
* [Installation](#installation)
* [How to use the puppet ansible module](#how-to-use-the-puppet-ansible-module)
  - [Deploy ansible](#deploy-ansible)
    - [Puppet side](#puppet-side)
      - [Plain text manifest](#plain-text-manifest)
      - [Hiera](#hiera)
    - [Ansible side](#ansible-side)
  - [Upgrade ansible](#upgrade-ansible)
    - [Latest version](#latest-version)
    - [Specific version](#specific-version)
* [Development](#development)
* [Documentation](#documentation)
* [Credits](#credits)
* [Licence](#licence)
* [References](#references)

## Definitions

In the following :

 - the **Ansible master** is the host where Ansible is installed and where you run your playbooks
 - the **Ansible nodes** are the hosts managed by the ansible master

## Description

The goals of the ansible puppet module are :

 - to install [Ansible](http://www.ansibleworks.com) on the ansible master
 - to allow ssh connections from the ansible master to a pool of ansible nodes
 - to install and configure sudo on the ansible nodes
The module use **public key authentication** and manage the **/etc/ssh/ssh_known_hosts** file of the ansible master.

On all hosts, an user **ansible** is created.

The ansible user on the master is able to run commands on the ansible nodes as **root** with **sudo**.

## Requirements

A puppet master with store config enable, because this module use exported ressources.

This module is created for Debian (Squeeze/Wheezy) and compatible with puppet agent (> 2.7).

This module use puppetlabs-stdlib (> 4.2.2).

The module has been tested on:

 - Debian 6
 - Debian 7

This module could be used on Ubuntu or CentOs (actually Debian and RedHat operating system family),
but tests has been light and support cannot be guaranteed.

## Installation and upgrade

From the forge, go to [nvogel/ansible](http://forge.puppetlabs.com/nvogel/ansible).

Or with [Librarian puppet](http://librarian-puppet.com/), for example add to your **Puppetfile** :

```
  mod 'ansible',
    :git => 'https://github.com/nvogel/puppet-ansible',
    :ref => '2.0.0'
```

Each version number follows the rules defined by [semantic versioning](http://semver.org/).

You should read the changelog file before upgrading to a new version and use only a tagged version with librarian.

## How to use the puppet ansible module

### Deploy ansible

#### Puppet side

On the ansible master with a fqdn **master.fqdn.tld**.

You can use hieara, an enc, or a plain text manifest.

You can have several ansible master hosts, each one will have its own pool of ansible nodes.

You have to wait 2 runs of the puppet agent to complete the configuration process.

Ansible is installed by default by pip, the provider parameter could be changed with apt.

##### Plain text manifest

```puppet
include ansible
```

or

```puppet
class { 'ansible':
  ensure => master
}
```

or if you want to use the apt provider

```puppet
include { 'ansible::master':
    provider => apt
}
```

For each ansible node :


```puppet
class { 'ansible':
  ensure => node,
  master => 'master.fqdn.tld'
}
```

or

```puppet
class { 'ansible::node' :
  master  => 'master.fqdn.tld'
}
```

##### Hiera

Example with a pool of hosts named **pool1**.

Each host have the same value for the fact **pool1**.

There is one host in the pool which is the ansible master (master.fqdn.tld).

**hiera.yaml** :

```yaml
---
:backends:
  - yaml
:yaml:
  :datadir: /etc/puppet/%{environment}/hieradata
:hierarchy:
  - "node/%{::clientcert}"
  - "pool/%{::pool}"
  - common
```

**hieradata directory** :

```
hieradata/
├── pool
│   └── pool1.yaml
└── node
    └── master.fqdn.tld.yaml
```

**pool1.yaml** :

```yaml
---
classes: ansible
ansible::ensure: node
ansible::master: master.fqdn.tld
```

**master.fqdn.tld.yaml** :

```yaml
---
ansible::ensure: master
ansible::master: false
```

#### Ansible side

On the ansible master host, all you have to do is to use the ansible user.
By default, the ansible user is set with a non valid password so you have to be root to use this account.

```bash
su - ansible
```

On the ansible nodes, the only package installed is **sudo**.
So, you may have to deploy with ansible additional python packages which are required for some ansible modules.

### Upgrade ansible

By default, puppet install the current version of ansible but will not upgrade it if already present.

You can change ansible version by setting the **ansible::install::version** parameter.

Example with hiera :

#### Latest version with apt

```yaml
---
ansible::ensure: master
ansible::master: false
ansible::master::provider: apt
ansible::install::version: latest
```

#### Specific version with pip

```yaml
---
ansible::ensure: master
ansible::master: false
ansible::install::version: 1.7.1
```

## Development

### Contributing

You're welcome to propose enhancements or submit bug reports (even typos).

When you perform modifications inside the puppet module :

 - You **MUST** run the test suite (see Testing section)
 - You **MUST** write (or update) the test suite
 - You **MUST** update the documentation

Thanks in advance.

### Branch management

 - Build status on branch master : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=master)](https://travis-ci.org/nvogel/puppet-ansible)
 - Build status on release 2.1.0 : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=v2.0.1)](https://travis-ci.org/nvogel/puppet-ansible)
 - Build status on release 2.0.1 : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=v2.0.1)](https://travis-ci.org/nvogel/puppet-ansible)
 - Build status on release 2.0.0 : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=v2.0.0)](https://travis-ci.org/nvogel/puppet-ansible)
 - Build status on release 1.1.1 : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=v1.1.1)](https://travis-ci.org/nvogel/puppet-ansible)
 - Build status on release 1.1.0 : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=v1.1.0)](https://travis-ci.org/nvogel/puppet-ansible)
 - Build status on release 1.0.0 : [![Build Status](https://travis-ci.org/nvogel/puppet-ansible.png?branch=v1.0.0)](https://travis-ci.org/nvogel/puppet-ansible)

The master branch corresponds to the release under development. Could be unstable.
All stable release are tagged.

### Installation

```bash
gem install bundler
mkdir modules
cd modules
git clone git://github.com/nvogel/puppet-ansible.git ansible
cd ansible
bundle install --path vendor/bundle
```

### Testing
#### How to list the available tasks

```bash
bundle exec rake
```

#### How to run syntax, lint and rspec

```bash
bundle exec rake test
```
#### How to check the syntax

```bash
bundle exec rake syntax
```
#### How to lint

```bash
bundle exec rake lint
```

#### How to run rspec on the manifests

```bash
bundle exec rake spec
```

#### How to run test automatically when you change any of the manifest

```bash
bundle exec guard
```
## Documentation
### How to generate the documentation of the module

```bash
mkdir -p /tmp/doc/ansible && touch /tmp/doc/manifest
cd /path/to/module/directory/ansible
ln -s "$(pwd)/lib" /tmp/doc/ansible
ln -s "$(pwd)/manifests" /tmp/doc/ansible
#generate module documentation in /path/to/ansible_doc from /tmp/doc
bundle exec puppet doc --charset UTF-8 --outputdir /path/to/ansible_doc --mode rdoc --manifest /tmp/doc/manifest --modulepath /tmp/doc
```

## Credits

* Nicolas Vogel
* [All contributors](https://github.com/nvogel/puppet-ansible/contributors)

## Licence

Puppet ansible module is released under the MIT License. Check the LICENSE file for details.

## References

- [puppet](http://puppetlabs.com)
- [ansible](http://www.ansibleworks.com)
- [contributing to open-source](https://guides.github.com/activities/contributing-to-open-source)
- [about commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
