# gitcmdb
## What is it?

CMDB system where CMDB server is git server (for example GitHub)

Every defined number of seconds, hosts connect to Github to update a specific branch of a repo and execute a list of installers only in case this branch has been updated.

Currently the only supported installers are puppet and simple bash scripts. New installers can be added easily.

Multiple installers can be executed in the same host and at the same cron execution sequence. So you can mix for example bash for some tasks and puppet for other more complex tasks.


## Installation
### Github preparation (to be done only once when creating your own repos)
* [Duplicate](https://help.github.com/articles/duplicating-a-repository/) both midonet/gitcmdb and midonet/github-data to your organization as private repos
* Create a Github account for bot usage and add this user as readonly collaborator or in a readonly team to those two repos
* Update in your gitcmdb repo your file `configs/default.yaml` to set your bot credentials in installerrepo variable like this:
```
...
installerrepo: https://USER:PASS@github.com/ORGANIZATION/gitcmdb-data
...
```
* Optional: update this README.md file in your repo to update next section credentials to your bot credentials, so you can read this readme and Copy&Paste.
### Server installation (to be done in all servers only once when enrolling to gitcmdb)
```
wget --quiet 'https://raw.githubusercontent.com/midonet/gitcmdb/master/gitcmdb?' -O - | bash -s https://USER:PASSWORD@github.com/ORGANIZATION/gitcmdb
```
where:
* ORGANIZATION is your Github organization
* USER is your bot username
* PASSWORD is yout bot password

Supported Distributions:
* Debian/Ubuntu
* CentOS/RHEL

## Usage
When creating hosts, define hostnames like *role*-*instance*-*whatever*.*domain*

Configuration variables will be load from in your github repo (hiera like):
* config/default.yaml
* config/*role*/default.yaml
* config/*role*/*instance*.yaml

and it that order so the later has preference

All those environment variables will be load and then *gitcmdb_installers* will be executed in the written order

## Specific installers
### Puppet
It's quite similar to r10k usage, every host has an specific repo (gitcmdb_installerrepo), an specific branch (gitcmdb_branch) and:
* Puppetfile is read by r10k to install all defined modules
* hiera is enabled, hiera.yaml is also versioned for every repository branch
* puppet apply is executed to load manifests/site.pp
* By default hiera loads class ::roles::*role*
* External facts from `facts.d` are linked to `/etc/facter/facts.d` so they can also be used

### Bash
All variables in yaml config will be loaded as environment variables.
If found those scripts are executed in that order:
* bash/scripts/default.sh
* bash/scripts/*role*.sh
* bash/scripts/*role*/*instance*.sh

Make idempotent scripts!

### Ansible
To be done.
