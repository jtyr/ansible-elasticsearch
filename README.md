elasticsearch
=============

Role which installs and configures Elasticsearch.

The configuration of the role is done in such way that it should not be necessary
to change the role for any kind of configuration. All can be done either by
changing role parameters or by declaring completely new configuration as a
variable. That makes this role absolutely universal. See the examples below for
more details.

Please report any issues or send PR.


Example
-------

```yaml
---

# Example of how to use the role with default configuration
- hosts: myhost1
  roles:
    - elasticsearch

# Example of how to customize the configuration
- hosts: myhost2
  vars:
    # Bind the process to a specific interface
    elasticsearch_config_network_host: _eth0_
    # Set JVM heap size
    elasticsearch_jvm_options_x_ms: 1g
    elasticsearch_jvm_options_x_mx: 1g
    # Setup cluster
    elasticsearch_config__custom:
      cluster:
        name: production
      node:
        name: "{{ inventory_hostname }}"
      discovery:
        zen:
          minimum_master_nodes: 2
          ping:
            unicast:
              hosts:
                - server1
                - server2
                - server3
      # Required for CentOS 6
      #bootstrap:
      #   system_call_filter: false
  roles:
    - elasticsearch

# Example of how to add Elasticsearch scripts, plugins and pipeline
- hosts: myhost3
  vars:
    # Enable scripting
    elasticsearch_config__custom:
      script:
        inline: "true"
        indexed: "true"
    # Create script files
    elasticsearch_scripts:
      # This is the filename
      calculate-score.groovy: |-2
        log(_score * 2) + my_modifier
      # Another script
      my_script.groovy: |-2
        1 + my_var
    # Install plugins
    elasticsearch_plugins:
      - x-pack
    # Create pipeline
    elasticsearch_pipeline_auth:
      user: elastic
      password: changeme
      # Whether to validate SSL cert
      validate_certs: no
    elasticsearch_pipelines:
      # Name of the pipeline
      apache:
        # Content of the pipeline
        description: Parse Apache logs
        processors:
          - grok:
              field: message
              patterns:
                - "%{COMBINEDAPACHELOG}"
  roles:
    - elasticsearch
```

The `logging.yml` file is not templated yet. Please let me know if you need
it to be templatable and I will add it.


Role variables
--------------

List of variables used by the role:

```yaml
# YUM repo URL
elasticsearch_yum_repo_url: "{{ elastic_yum_repo_url | default('https://artifacts.elastic.co/packages/6.x/yum') }}"

# YUM repo GPG key
elasticsearch_yum_repo_key: "{{ elastic_yum_repo_key | default('https://artifacts.elastic.co/GPG-KEY-elasticsearch') }}"

# Additional YUM repo parameters
elasticsearch_yum_repo_params: "{{ elastic_yum_repo_params | default({}) }}"

# GPG key for the APT repo
elasticsearch_apt_repo_key: "{{ elastic_apt_repo_key | default('https://artifacts.elastic.co/GPG-KEY-elasticsearch') }}"

# APT repo string
elasticsearch_apt_repo_string: "{{ elastic_apt_repo_string | default('deb https://artifacts.elastic.co/packages/6.x/apt stable main') }}"

# Additional APT repo parameters
elasticsearch_apt_repo_params: "{{ elastic_apt_repo_params | default({}) }}"

# Package to be installed (explicit version can be specified here)
elasticsearch_pkg: elasticsearch

# Additional packages to be installed (e.g. Java)
elasticsearch_additional_pkgs:
  - "{{
        'java'
          if ansible_facts.os_family == 'RedHat'
          else
        'openjdk-8-jdk' }}"

# Name of the service
elasticsearch_service: elasticsearch


# Path to the elasticsearch-plugin executable
elasticsearch_plugins_bin: /usr/share/elasticsearch/bin/elasticsearch-plugin

# Plugin installation flags
elasticsearch_plugins_flags: --batch

# List of plugins to install
elasticsearch_plugins: []


# How through which to manage pipelines
elasticsearch_pipeline_host: 127.0.0.1

# Pipeline authentication
elasticsearch_pipeline_auth: null

# Pipelines
elasticsearch_pipelines: {}


# Path to the config file
elasticsearch_config_file: /etc/elasticsearch/elasticsearch.yml

# Value of the network options
elasticsearch_config_network_host: _local_

# Default network options
elasticsearch_config_network__default:
  host: "{{ elasticsearch_config_network_host }}"

# Custom network options
elasticsearch_config_network__custom: {}

# Final network options
elasticsearch_config_network: "{{
  elasticsearch_config_network__default | combine(
  elasticsearch_config_network__custom) }}"


# Value of the http options
elasticsearch_config_http_port: 9200

# Default http options
elasticsearch_config_http__default:
  port: "{{ elasticsearch_config_http_port }}"

# Custom http options
elasticsearch_config_http__custom: {}

# Final http options
elasticsearch_config_http: "{{
  elasticsearch_config_http__default | combine(
  elasticsearch_config_http__custom) }}"


# Value of the path options
elasticsearch_config_path_data: /var/lib/elasticsearch
elasticsearch_config_path_logs: /var/log/elasticsearch

# Default path options
elasticsearch_config_path__default:
  data: "{{ elasticsearch_config_path_data }}"
  logs: "{{ elasticsearch_config_path_logs }}"

# Custom path options
elasticsearch_config_path__custom: {}

# Final path options
elasticsearch_config_path: "{{
  elasticsearch_config_path__default | combine(
  elasticsearch_config_path__custom) }}"


# Default elasticsearch configuration
elasticsearch_config__default:
  network: "{{ elasticsearch_config_network }}"
  http: "{{ elasticsearch_config_http }}"
  path: "{{ elasticsearch_config_path }}"

# Custom elasticsearch configuration
elasticsearch_config__custom: {}

# Final elasticsearch configuration
elasticsearch_config: "{{
  elasticsearch_config__default | combine(
  elasticsearch_config__custom) }}"


# Protocol to use for the communication with the server
elasticsearch_protocol: "{{
  'https'
    if
      'xpack' in elasticsearch_config and
      'security' in elasticsearch_config.xpack and
      'http' in elasticsearch_config.xpack.security and
      'ssl' in elasticsearch_config.xpack.security.http and
      'enabled' in elasticsearch_config.xpack.security.http.ssl and
      elasticsearch_config.xpack.security.http.ssl.enabled == true
    else
  'http' }}"


# Path to the sysconfig file
elasticsearch_sysconfig_file: /etc/sysconfig/elasticsearch

# Vaules of the defautl sysconfig options
elasticsearch_sysconfig_es_startup_sleep_time: 5

# Defautl sysconfig options
elasticsearch_sysconfig__default:
  es_startup_sleep_time: "{{ elasticsearch_sysconfig_es_startup_sleep_time }}"
# Possible values:
#elasticsearch_sysconfig__default:
#  conf_dir: /etc/elasticsearch
#  data_dir: /var/lib/elasticsearch
#  es_group: elasticsearch
#  es_home: /usr/share/elasticsearch
#  es_java_opts: ""
#  es_user: elasticsearch
#  java_home: ""
#  log_dir: /var/log/elasticsearch
#  max_locked_memory: unlimited
#  max_map_count: 262144
#  max_open_files: 65536
#  pid_dir: /var/run/elasticsearch
#  restart_on_upgrade: "true"

# Custom sysconfig options
elasticsearch_sysconfig__custom: {}

# Final sysconfig options
elasticsearch_sysconfig: "{{
  elasticsearch_sysconfig__default | combine(
  elasticsearch_sysconfig__custom) }}"


# Path to the scripts directory
elasticsearch_scripts_path: /etc/elasticsearch/scripts

# Scripts
elasticsearch_scripts: {}


# Location of the jvm.options file
elasticsearch_jvm_options_file: /etc/elasticsearch/jvm.options

# Values of the default JVM system properties
elasticsearch_jvm_options_d_file_encoding: UTF-8
elasticsearch_jvm_options_d_io_netty_noKeySetOptimization: "true"
elasticsearch_jvm_options_d_io_netty_noUnsafe: "true"
elasticsearch_jvm_options_d_io_netty_recycler_maxCapacityPerThread: 0
elasticsearch_jvm_options_d_java_awt_headless: "true"
elasticsearch_jvm_options_d_jdk_io_permissionsUseCanonicalPath: "true"
elasticsearch_jvm_options_d_jna_nosys: "true"
elasticsearch_jvm_options_d_log4j2_disable_jmx: "true"
elasticsearch_jvm_options_d_log4j_shutdownHookEnabled: "false"
elasticsearch_jvm_options_d_log4j_skipJansi: "true"

# Default JVM system properties
elasticsearch_jvm_options_d__default:
  - file.encoding={{ elasticsearch_jvm_options_d_file_encoding }}
  - io.netty.noKeySetOptimization={{ elasticsearch_jvm_options_d_io_netty_noKeySetOptimization }}
  - io.netty.noUnsafe={{ elasticsearch_jvm_options_d_io_netty_noUnsafe }}
  - io.netty.recycler.maxCapacityPerThread={{ elasticsearch_jvm_options_d_io_netty_recycler_maxCapacityPerThread }}
  - java.awt.headless={{ elasticsearch_jvm_options_d_java_awt_headless }}
  - jdk.io.permissionsUseCanonicalPath={{ elasticsearch_jvm_options_d_jdk_io_permissionsUseCanonicalPath }}
  - jna.nosys={{ elasticsearch_jvm_options_d_jna_nosys }}
  - log4j2.disable.jmx={{ elasticsearch_jvm_options_d_log4j2_disable_jmx }}
  - log4j.shutdownHookEnabled={{ elasticsearch_jvm_options_d_log4j_shutdownHookEnabled }}
  - log4j.skipJansi={{ elasticsearch_jvm_options_d_log4j_skipJansi }}

# Custom JVM system properties
elasticsearch_jvm_options_d__custom: []

# Final JVM system properties
elasticsearch_jvm_options_d: "{{
  elasticsearch_jvm_options_d__default +
  elasticsearch_jvm_options_d__custom }}"


# Values of the default JVM non-standard options
elasticsearch_jvm_options_x_ms: 2g
elasticsearch_jvm_options_x_mx: 2g
elasticsearch_jvm_options_x_ss: 1m
elasticsearch_jvm_options_x_X_CMSInitiatingOccupancyFraction: 75
elasticsearch_jvm_options_x_X_AlwaysPreTouch: yes
elasticsearch_jvm_options_x_X_DisableExplicitGC: yes
elasticsearch_jvm_options_x_X_HeapDumpOnOutOfMemoryError: yes
elasticsearch_jvm_options_x_X_UseCMSInitiatingOccupancyOnly: yes
elasticsearch_jvm_options_x_X_UseConcMarkSweepGC: yes

# Default JVM non-standard options
elasticsearch_jvm_options_x__default:
  - ms{{ elasticsearch_jvm_options_x_ms }}
  - mx{{ elasticsearch_jvm_options_x_mx }}
  - ss{{ elasticsearch_jvm_options_x_ss }}
  - X:CMSInitiatingOccupancyFraction={{ elasticsearch_jvm_options_x_X_CMSInitiatingOccupancyFraction }}
  - X:{{ '+' if elasticsearch_jvm_options_x_X_AlwaysPreTouch else '-' }}AlwaysPreTouch
  - X:{{ '+' if elasticsearch_jvm_options_x_X_DisableExplicitGC else '-' }}DisableExplicitGC
  - X:{{ '+' if elasticsearch_jvm_options_x_X_HeapDumpOnOutOfMemoryError else '-' }}HeapDumpOnOutOfMemoryError
  - X:{{ '+' if elasticsearch_jvm_options_x_X_UseCMSInitiatingOccupancyOnly else '-' }}UseCMSInitiatingOccupancyOnly
  - X:{{ '+' if elasticsearch_jvm_options_x_X_UseConcMarkSweepGC else '-' }}UseConcMarkSweepGC

# Custom JVM non-standard options
elasticsearch_jvm_options_x__custom: []

# Final JVM non-standard options
elasticsearch_jvm_options_x: "{{
  elasticsearch_jvm_options_x__default +
  elasticsearch_jvm_options_x__custom }}"


# Default JVM options
elasticsearch_jvm_options__default:
  -server: " "
  -D: "{{ elasticsearch_jvm_options_d }}"
  -X: "{{ elasticsearch_jvm_options_x }}"

# Custom JVM options
elasticsearch_jvm_options__custom: {}

# Final JVM options
elasticsearch_jvm_options: "{{
  elasticsearch_jvm_options__default | combine(
  elasticsearch_jvm_options__custom) }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`filebeat`](https://github.com/jtyr/ansible-filebeat) (optional)
- [`logstash`](https://github.com/jtyr/ansible-logstash) (optional)
- [`kibana`](https://github.com/jtyr/ansible-kibana) (optional)
- [`oracle-java`](https://github.com/jtyr/ansible-oracle_java) (optional)


License
-------

MIT


Author
------

Jiri Tyr
