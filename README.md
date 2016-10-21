elasticsearch
=============

Role which installs and configures Elasticsearch.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Example
-------

```
---

# Example of how to use the role with default configuration
- hosts: myhost1
  roles:
    - elasticsearch

# Example of how to customize the configuration
- hosts: myhost2
  vars:
    # Set binding to the main interface
    elasticsearch_config:
      network.bind_host: "{{ ansible_eth0.ipv4.address }}"
      http.host: "{{ ansible_eth0.ipv4.address }}"
    # Set new JVM heap size
    elasticsearch_sysconfig:
      es_heap_size: 512m
  roles:
    - elasticsearch
```


Role variables
--------------

List of variables used by the role:

```
# YUM repo URL
elasticsearch_yum_repo_url: https://packages.elastic.co/elasticsearch/2.x/centos/

# YUM repo GPG key
elasticsearch_yum_repo_key: https://packages.elastic.co/GPG-KEY-elasticsearch

# Additional YUM repo parameters
elasticsearch_yum_repo_params: {}

# Package to be installed (explicit version can be specified here)
elasticsearch_pkg: elasticsearch

# Additional packages to be installed (e.g. Java)
elasticsearch_additional_pkgs:
  - java

# Name of the service
elasticsearch_service: elasticsearch

# Path to the config file
elasticsearch_config_file: /etc/elasticsearch/elasticsearch.yml


# Value of the bind_host option
elasticsearch_config_network_bind_host: 127.0.0.1

# Default network options
elasticsearch_config_network__default:
  bind_host: "{{ elasticsearch_config_network_bind_host }}"

# Custom network options
elasticsearch_config_network__custom: {}

# Final network options
elasticsearch_config_network: "{{
  elasticsearch_config_network__default.update(
  elasticsearch_config_network__custom) }}{{
  elasticsearch_config_network__default }}"


# Value of the bind_host option
elasticsearch_config_http_host: 127.0.0.1

# Default network options
elasticsearch_config_http__default:
  host: "{{ elasticsearch_config_http_host }}"

# Custom network options
elasticsearch_config_http__custom: {}

# Final network options
elasticsearch_config_http: "{{
  elasticsearch_config_http__default.update(
  elasticsearch_config_http__custom) }}{{
  elasticsearch_config_http__default }}"


# Default elasticsearch configuration
elasticsearch_config:
  network: "{{ elasticsearch_config_network }}"
  http: "{{ elasticsearch_config_http }}"


# Path to the sysconfig file
elasticsearch_sysconfig_file: /etc/sysconfig/elasticsearch

# Sysconfig file configuration
elasticsearch_sysconfig:
  es_startup_sleep_time: 5
# Possible options:
#elasticsearch_sysconfig:
#  es_home: /usr/share/elasticsearch
#  conf_dir: /etc/elasticsearch
#  data_dir: /var/lib/elasticsearch
#  log_dir: /var/log/elasticsearch
#  pid_dir: /var/run/elasticsearch
#  es_heap_size: 2g
#  es_heap_newsize: ""
#  es_direct_size: ""
#  es_java_opts: ""
#  restart_on_upgrade: "true"
#  es_gc_log_file: /var/log/elasticsearch/gc.log
#  es_user: elasticsearch
#  es_group: elasticsearch
#  es_startup_sleep_time: 5
#  max_open_files: 65536
#  max_locked_memory: unlimited
#  max_map_count: 262144
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`oracle-java`](https://github.com/jtyr/ansible-oracle_java) (optional)


License
-------

MIT


Author
------

Jiri Tyr
