# chef-stash  [![Build Status](https://secure.travis-ci.org/bflad/chef-stash.png?branch=master)](http://travis-ci.org/bflad/chef-stash)

## Description

Installs/Configures [Atlassian Stash](https://www.atlassian.com/software/stash/) server and [Atlassian Stash Backup Client](https://marketplace.atlassian.com/plugins/com.atlassian.stash.backup.client). Provides LWRPs for code deployment via Stash as well as for hook and repository management. Please see [COMPATIBILITY.md](COMPATIBILITY.md) for more information about Stash releases (versions and checksums) that are tested and supported by cookbook versions.

## Requirements

### Platforms

* CentOS 6
* RHEL 6
* Ubuntu 12.04, 12.10, 13.04

### Chef

* Version 3.X of cookbook requires Chef 11 due to `ark` usage

### Databases

* HSQLDB (not recommended for production usage)
* Microsoft SQL Server
* MySQL
* Postgres

### Cookbooks

Required [Opscode Cookbooks](https://github.com/opscode-cookbooks/)

* [apache2](https://github.com/opscode-cookbooks/apache2) (if using Apache 2 proxy)
* [ark](https://github.com/opscode-cookbooks/ark)
* [cron](https://github.com/opscode-cookbooks/cron)
* [database](https://github.com/opscode-cookbooks/database)
* [git](https://github.com/opscode-cookbooks/git)
* [java](https://github.com/opscode-cookbooks/java)
* [mysql](https://github.com/opscode-cookbooks/mysql) (if using MySQL database)
* [perl](https://github.com/opscode-cookbooks/perl)
* [postgresql](https://github.com/opscode-cookbooks/postgresql) (if using Postgres database)

Third-Party Cookbooks

* [mysql_connector](https://github.com/bflad/chef-mysql_connector) (if using MySQL database)

## Attributes

These attributes are under the `node['stash']` namespace.

Attribute | Description | Type | Default
----------|-------------|------|--------
checksum | SHA256 checksum for Stash install | String | auto-detected (see attributes/default.rb)
home_path | home data directory for Stash user | String | /var/atlassian/application-data/stash
install_path | location to install Stash | String | /opt/atlassian
install_type | Stash install type - "standalone" only for now | String | standalone
url_base | URL base for Stash install | String | http://www.atlassian.com/software/stash/downloads/binary/atlassian-stash
url | URL for Stash install | String | auto-detected (see attributes/default.rb)
user | user to run Stash | String | stash
version | Stash version to install | String | 2.9.4

### Stash Backup Client Attributes

These attributes are under the `node['stash']['backup_client']` namespace. Some of these attributes are overridden by `stash/stash` encrypted data bag (Hosted Chef) or data bag (Chef Solo), if it exists.

Attribute | Description | Type | Default
----------|-------------|------|--------
backup_path | Path for backups | String | /tmp
baseurl | Stash base URL | String | `https://#{node['fqdn']}/`
checksum | SHA256 checksum for Stash Backup Client install | String | auto-detected (see attributes/default.rb)
install_path | location to install Stash Backup Client | String | /opt/atlassian-stash-backup-client
password | Stash administrative user password | String | changeit
url_base | URL base for Stash Backup Client install | String | http://downloads.atlassian.com/software/stash/downloads/stash-backup-distribution
user | Stash administrative user | String | admin
version | Stash Backup Client version to install | String | 1.0.3

### Stash Backup Client Cron Attributes

These attributes are under the `node['stash']['backup_client']['cron']` namespace. All of these attributes are overridden by `stash/stash` encrypted data bag (Hosted Chef) or data bag (Chef Solo), if it exists.

Attribute | Description | Type | Default
----------|-------------|------|--------
day | Day of month | String | *
hour | Hour of day | String | 0
minute | Minute of hour | String | 0
month | Month of year | String | *
weekday | Day of week | String | *

### Stash Database Attributes

All of these `node['stash']['database']` attributes are overridden by `stash/stash` encrypted data bag (Hosted Chef) or data bag (Chef Solo), if it exists

Attribute | Description | Type | Default
----------|-------------|------|--------
host | FQDN or "localhost" (localhost automatically installs `['database']['type']` server) | String | localhost
name | Stash database name | String | stash
password | Stash database user password | String | changeit
port | Stash database port | Fixnum | 3306
testInterval | Stash database pool idle test interval in minutes | Fixnum | 2
type | Stash database type - "hsqldb" (not recommended), "mysql", "postgresql", or "sqlserver" | String | mysql
user | Stash database user | String | stash

### Stash JVM Attributes

These attributes are under the `node['stash']['jvm']` namespace.

Attribute | Description | Type | Default
----------|-------------|------|--------
minimum_memory | JVM minimum memory | String | 512m
maximum_memory | JVM maximum memory | String | 768m
maximum_permgen | JVM maximum PermGen memory | String | 256m
java_opts | additional JAVA_OPTS to be passed to Stash JVM during startup | String | ""
support_args | additional JAVA_OPTS recommended by Atlassian support for Stash JVM during startup | String | ""

### Stash Plugin Attributes

All of these `node['stash']['plugin']` attributes are overridden by `stash/stash` encrypted data bag (Hosted Chef) or data bag (Chef Solo), if it exists

Attribute | Description | Type | Default
----------|-------------|------|--------
`key` | A key/value pair to be inserted into stash-config.properties as plugin.`key`=`value` | - | -

### Stash SSH Attributes ###

These attributes are under the `node['stash']['ssh']` namespace.

Attribute | Description | Type | Default
----------|-------------|------|--------
hostname | Stash SSH hostname | String | `node['fqdn']`
port | Stash SSH port | Fixnum | 7999
uri | Stash SSH URI | String | `ssh://git@#{node['stash']['ssh']['hostname']}:#{node['stash']['ssh']['port']}`

### Stash Tomcat Attributes

These attributes are under the `node['stash']['tomcat']` namespace.

Any `node['stash']['tomcat']['key*']` attributes are overridden by `stash/stash` encrypted data bag (Hosted Chef) or data bag (Chef Solo), if it exists

Attribute | Description | Type | Default
----------|-------------|------|--------
keyAlias | Tomcat SSL keystore alias | String | tomcat
keystoreFile | Tomcat SSL keystore file - will automatically generate self-signed keystore file if left as default | String | `#{node['stash']['home_path']}/.keystore`
keystorePass | Tomcat SSL keystore passphrase | String | changeit
port | Tomcat HTTP port | Fixnum | 7990
ssl_port | Tomcat HTTPS port | Fixnum | 8443

## Recipes

* `recipe[stash]` Installs Atlassian Stash with built-in Tomcat and Apache 2 proxy
* `recipe[stash::apache2]` Installs/configures Apache 2 proxy for Stash (ports 80/443)
* `recipe[stash::backup_client]` Installs/configures Atlassian Stash Backup Client
* `recipe[stash::backup_client_cron]` Installs/configures Atlassian Stash Backup Client cron.d
* `recipe[stash::configuration]` Configures Stash's settings
* `recipe[stash::database]` Installs/configures MySQL/Postgres server, database, and user for Stash
* `recipe[stash::linux_standalone]` Installs/configures Stash via Linux standalone archive
* `recipe[stash::service_init]` Installs/configures Stash init service
* `recipe[stash::tomcat_configuration]` Configures Stash's built-in Tomcat

## LWRPs

* `stash_deploy` - wrapper Git resource for using a `stash_deploy_key`, project, and repository for code deployment
* `stash_deploy_key` - creates SSH private key file and SSH wrapper for code deployment
* `hook` - Wrapper to enable/disable/configure a stash hook (requires the user account password to be in chef-vault)
* `repo` - Wrapper to create/delete a stash repository (requires the user account password to be in chef-vault)

## Usage

### Stash Server Data Bag

For securely overriding attributes on Hosted Chef, create a `stash/stash` encrypted data bag with the model below. Chef Solo can override the same attributes with a `stash/stash` unencrypted data bag of the same information.

_required:_
* `['database']['type']` "hsqldb" (not recommended), "mysql", "postgresql", or "sqlserver"
* `['database']['host']` FQDN or "localhost" (localhost automatically
  installs `['database']['type']` server)
* `['database']['name']` Name of Stash database
* `['database']['user']` Stash database username
* `['database']['password']` Stash database username password

_optional:_
* `['backup_client']['user']` Stash administrative username for backup client
* `['backup_client']['password']` Stash administrative password for backup client
* `['database']['port']` Database port, standard database port for
  `['database']['type']`
* `['plugin']['KEY']` plugin.`KEY`=`VALUE` to be inserted in stash-config.properties
* `['tomcat']['keyAlias']` Tomcat HTTPS Java Keystore keyAlias, defaults to self-signed certifcate
* `['tomcat']['keystoreFile']` Tomcat HTTPS Java Keystore keystoreFile, self-signed certificate
* `['tomcat']['keystorePass']` Tomcat HTTPS Java Keystore keystorePass, self-signed certificate

Repeat for other Chef environments as necessary. Example:

    {
      "id": "stash"
      "development": {
        "database": {
          "type": "postgresql",
          "host": "localhost",
          "name": "stash",
          "user": "stash",
          "password": "stash_db_password",
        },
        "tomcat": {
          "keyAlias": "not_tomcat",
          "keystoreFile": "/etc/pki/java/wildcard_cert.jks",
          "keystorePass": "not_changeit"
        }
      }
    }

### Stash Server Default Installation

* Optionally use (un)encrypted data bag or set attributes
  * `knife data bag create stash`
  * `knife data bag edit stash stash --secret-file=path/to/secret`
* Add `recipe[stash]` to your node's run list.

### Stash Backup Client Installation

* Optionally use (un)encrypted data bag or set attributes
  * `knife data bag create stash`
  * `knife data bag edit stash stash --secret-file=path/to/secret`
* Add `recipe[stash]['backup_client']` to your node's run list.

### Stash Backup Client Cron Installation

* Optionally use (un)encrypted data bag or set attributes
  * `knife data bag create stash`
  * `knife data bag edit stash stash --secret-file=path/to/secret`
* Add `recipe[stash]['backup_client_cron']` to your node's run list.

### Code Deployment From Stash

* Ensure your node has Git installed
* Create a `stash_deploy_key` with the SSH private key contents (using `\n` for newlines) of a Stash user with permissions to your repository.

For example:

    stash_deploy_key "deployment_user" do
      key "-----BEGIN RSA PRIVATE KEY-----\nMIIEpQIB..."
    end

* In this example, now you can either directly use the ssh_wrapper available at `#{node['stash']['install_path']}/deployment_user_ssh_wrapper.sh` or use the `stash_deploy` LWRP. 

Such as:

    stash_deploy "/opt/shibboleth-idp/conf" do
      deploy_key "deployment_user"
      project "SHIBIDP"
      repository "configuration"
    end

## Testing and Development

Here's how you can quickly get testing or developing against the cookbook thanks to [Vagrant](http://vagrantup.com/) and [Berkshelf](http://berkshelf.com/).

    vagrant plugin install vagrant-berkshelf
    vagrant plugin install vagrant-cachier
    vagrant plugin install vagrant-omnibus
    git clone git://github.com/bflad/chef-stash.git
    cd chef-stash
    vagrant up BOX # BOX being centos5, centos6, debian7, fedora18, fedora19, fedora20, freebsd9, ubuntu1204, ubuntu1210, ubuntu1304, or ubuntu1310

You need to add the following hosts entries:

* 192.168.50.10 stash-centos-6
* 192.168.50.10 stash-ubuntu-1204
* (etc.)

The running Stash server is accessible from the host machine:

CentOS 6 Box:
* Web UI: https://stash-centos-6/
* Stash SSH: ssh://git@stash-centos-6:7999/

Ubuntu 12.04 Box:
* Web UI: https://stash-ubuntu-1204/
* Stash SSH: ssh://git@stash-ubuntu-1204:7999/

You can then SSH into the running VM using the `vagrant ssh BOX` command.

The VM can easily be stopped and deleted with the `vagrant destroy` command. Please see the official [Vagrant documentation](http://docs.vagrantup.com/v2/cli/index.html) for a more in depth explanation of available commands.

### Test Kitchen

Please see documentation in: [TESTING.md](TESTING.md)

## Contributing

Please use standard Github issues/pull requests and if possible, in combination with testing on the Vagrant boxes.

## License and Contributors

Please see license information in: [LICENSE](LICENSE)

* Brian Flad (<bflad417@gmail.com>)
* Kevin Moser
* @ramonskie
