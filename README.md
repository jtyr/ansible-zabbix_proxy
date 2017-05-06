zabbix_proxy
============

Ansible role which helps to install and configure Zabbix Proxy.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
- name: Example of default with most basic configuration
  hosts: machine1
  vars:
    # IP (or hostname) of the server
    zabbix_proxy_config_server: 10.0.0.122
  roles:
    - postgresql
    - zabbix_proxy

- name: Example of how to change default DB connection details
  hosts: machine2
  vars:
    # IP (or hostname) of the server
    zabbix_proxy_config_server: 10.0.0.122
    # Change engine to MySQL
    zabbix_proxy_db_engine: mysql
    # The DB is running on a remote machine
    zabbix_proxy_db_host: 10.0.0.123
    # Admin DB credentials for the zabbix user creation
    zabbix_proxy_db_login_user: admin
    zabbix_proxy_db_login_password: s3cr3t
    # Zabbix DB user credentials
    zabbix_proxy_db_user: zabbix
    zabbix_proxy_db_password: z4b1x123
  roles:
    - mysql
    - zabbix_proxy

- name: Example of how to change default Logging configuration
  hosts: machine3
  vars:
    # IP (or hostname) of the server
    zabbix_proxy_config_server: 10.0.0.122
    # Increase the default log file size for log rotation to 10MB
    zabbix_proxy_logging_filesize: 10
    # Add custom Logging option
    zabbix_proxy_logging__custom:
      # Change the debug level to "Critical information"
      DebugLevel: 1
      # Change frequency of sending unsent alerts from 30 to 5 secs
      SenderFrequency: 5
      # Set timeout to 10 seconds
      Timeout: 10
  roles:
    - zabbix_proxy

- name: Example of how to write zabbix_proxy.conf from scratch
  hosts: machine4
  vars:
    zabbix_proxy_config:
      Server: 10.0.0.122
      DBName: zabbix
      DebugLevel: 1
      Timeout: 10
  roles:
    - zabbix_proxy
```

The default DB engine is set to PostgreSQL but it can be changed to MySQL
(see example above). The change of the engine must be done
before the first run of the role.


Role variables
--------------

```
# Major Zabbix version (used for Zabbix YUM repo)
zabbix_proxy_major_version: "{{ zabbix_major_version | default(2.4) }}"

# Zabbix YUM repo URL
zabbix_proxy_yumrepo_url: "{{ zabbix_yumrepo_url | default('http://repo.zabbix.com/zabbix/' ~ zabbix_proxy_major_version ~ '/rhel/' ~ ansible_distribution_major_version ~ '/$basearch/') }}"

# Additional Zabbix YUM repo params
zabbix_proxy_yumrepo_params: "{{ zabbix_yumrepo_params | default({}) }}"

# Path to the zabix_proxy.conf file
zabbix_proxy_config_file: /etc/zabbix/zabbix_proxy.conf

# Whether to install EPEL YUM repo
zabbix_proxy_epel_install: "{{ yumrepo_epel_install | default(true) }}"

# EPEL YUM repo URL
zabbix_proxy_epel_yumrepo_url: "{{ yumrepo_epel_url | default('https://dl.fedoraproject.org/pub/epel/$releasever/$basearch/') }}"

# EPEL YUM repo GPG key
zabbix_proxy_epel_yumrepo_gpgkey: "{{ yumrepo_epel_gpgkey | default('https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever') }}"

# Additional EPEL YUM repo params
zabbix_proxy_epel_yumrepo_params: "{{ yumrepo_epel_params | default({}) }}"

# DB engine
zabbix_proxy_db_engine: pgsql

# Package to be installed (exact version can be specified here)
zabbix_proxy_pkg: zabbix-proxy-{{ zabbix_proxy_db_engine }}

# Default PostgreSQL port
zabbix_proxy_db_port_pgsql_default: 5432

# Default MySQL port
zabbix_proxy_db_port_mysql_default: 3306

# Initial DB schema directory
zabbix_proxy_db_schema_dir: /usr/share/doc/zabbix-proxy-{{ zabbix_proxy_db_engine }}-*

# User used to create DB and user
zabbix_proxy_db_login_user: "{{
  'postgres'
    if zabbix_proxy_db_engine == 'pgsql'
    else
  'root' }}"

# Password used to create DB and user
zabbix_proxy_db_login_password: "{{
  'postgres'
     if zabbix_proxy_db_engine == 'pgsql'
     else
  None }}"

# Packages required for the PostgreSQL DB creation
zabbix_proxy_db_pkgs_pgsql:
  - python-psycopg2
  - postgresql

# Packages required for the MySQL DB creation
zabbix_proxy_db_pkgs_mysql:
  - MySQL-python
  - mysql

zabbix_proxy_db_pkgs: "{{
  zabbix_proxy_db_pkgs_pgsql
    if zabbix_proxy_db_engine == 'pgsql'
    else
  zabbix_proxy_db_pkgs_mysql }}"


# DB server configuration options
zabbix_proxy_db_host: localhost
zabbix_proxy_db_port: "{{
  zabbix_proxy_db_port_pgsql_default
    if zabbix_proxy_db_engine == 'pgsql'
    else
  zabbix_proxy_db_port_mysql_default }}"
zabbix_proxy_db_name: zabbix
zabbix_proxy_db_user: zabbix
zabbix_proxy_db_password: zabbix


# Values of default configuration options
zabbix_proxy_config_server: 127.0.0.1
zabbix_proxy_config_hostname: "{{ inventory_hostname_short }}"
zabbix_proxy_config_logfile: /var/log/zabbix/zabbix_proxy.log
zabbix_proxy_config_logfilesize: 1
zabbix_proxy_config_pidfile: /var/run/zabbix/zabbix_proxy.pid
zabbix_proxy_config_externalscripts: /usr/lib/zabbix/externalscripts
zabbix_proxy_config_logslowqueries: 3000

# Default configuration options
zabbix_proxy_config__default:
  Server: "{{ zabbix_proxy_config_server }}"
  Hostname: "{{ zabbix_proxy_config_hostname }}"
  DBHost: "{{ zabbix_proxy_db_host }}"
  DBPort: "{{ zabbix_proxy_db_port }}"
  DBName: "{{ zabbix_proxy_db_name }}"
  DBUser: "{{ zabbix_proxy_db_user }}"
  DBPassword: "{{ zabbix_proxy_db_password }}"
  LogFile: "{{ zabbix_proxy_config_logfile }}"
  LogFileSize: "{{ zabbix_proxy_config_logfilesize }}"
  PidFile: "{{ zabbix_proxy_config_pidfile }}"
  ExternalScripts: "{{ zabbix_proxy_config_externalscripts }}"
  LogSlowQueries: "{{ zabbix_proxy_config_logslowqueries }}"

# Custom Zabbix configuration
zabbix_proxy_config__custom: {}

# Main Zabbix config
zabbix_proxy_config: "{{
  zabbix_proxy_config__default.update(zabbix_proxy_config__custom) }}{{
  zabbix_proxy_config__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`mysql`](http://github.com/jtyr/ansible-mysql) (optional)
- [`postgresql`](http://github.com/jtyr/ansible-postgresql) (optional)
- [`zabbix_agent`](https://github.com/jtyr/ansible-zabbix_agent) (optional)
- [`zabbix_server`](https://github.com/jtyr/ansible-zabbix_server) (optional)
- [`zabbix_web`](https://github.com/jtyr/ansible-zabbix_web) (optional)


License
-------

MIT


Author
------

Jiri Tyr
