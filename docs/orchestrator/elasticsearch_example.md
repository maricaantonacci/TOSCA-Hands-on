### [◀](../README.md)

# Installing Elasticsearch and Kibana with ansible

``` yaml tab="Elasticsearch playbook"
- name: Simple Elasticsearch Example
  hosts: localhost
  roles:
    - role: elastic.elasticsearch
  vars:
    es_version: 7.4.1
    es_config:
      network.bind_host: 0.0.0.0
      discovery.type: single-node
    es_enable_xpack: true
    es_api_basic_auth_username: elastic
    es_api_basic_auth_password: 12qwas
    es_users:
      native:
        kibana:
```

``` yaml tab="Kibana playbook"
- name: Simple Kibana Example
  hosts: localhost
  roles:
    - role: geerlingguy.kibana
  vars:
    kibana_version: 7.4.1
    kibana_elasticsearch_username: kibana
    kibana_elasticsearch_password: 12qwas
```

# Elasticsearch template

``` yaml tab="tosca template excerpt" hl_lines="2"
    elasticsearch:
      type: tosca.nodes.indigo.Elasticsearch
      properties:
        es_version:  { get_input: es_version }
        bind_address: { get_input: es_bind_address }
        elastic_password: { get_input: es_password }
        kibana_system_password: { get_input: kibana_password }
      requirements:
        - host: server
```

``` yaml tab="tosca custom type" hl_lines="18 23"
  tosca.nodes.indigo.Elasticsearch:
    derived_from: tosca.nodes.SoftwareComponent
    properties:
      es_version:
        type: string
        required: false
        default: 7.4.1
      bind_address:
        type: string
        required: false
        default: 0.0.0.0
      discovery_type:
        type: string
        required: false
        default: single-node
      enable_security:
        type: boolean
        default: true
        required: false
      elastic_password:
        type: string
        required: false
        default: changeme
      kibana_system_password:
        type: string
        required: false
        default: changeme
    artifacts:
      es_role:
        file: elastic.elasticsearch,7.4.1
        type: tosca.artifacts.AnsibleGalaxy.role
    interfaces:
      Standard:
        configure:
          implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/k8s_add_es/artifacts/elk/elasticsearch_install.yml
          inputs:
            es_version: { get_property: [ SELF, es_version ] }
            bind_host: { get_property: [ SELF, bind_address ] }
            discovery_type: { get_property: [ SELF, discovery_type ] }
            enable_security: { get_property: [ SELF, enable_security ] }
            elastic_password: { get_property: [ SELF, elastic_password ] }
            kibana_system_password: { get_property: [ SELF, kibana_system_password ] }
```

``` yaml tab="ansible artefact"
---
- hosts: localhost
  connection: local
  vars:
    es_config:
      network.bind_host: "{{ bind_host }}"
      discovery.type: "{{ discovery_type }}"
    es_enable_xpack: "{{ enable_security }}"
    es_api_basic_auth_username: elastic
    es_api_basic_auth_password: "{{ elastic_password  }}"
    es_users:
      native:
        kibana:
          password: "{{ kibana_system_password }}"
  roles:
    - role: elastic.elasticsearch
```

## Add Kibana

``` yaml tab="tosca template excerpt" hl_lines="2"
    kibana:
      type: tosca.nodes.indigo.Kibana
      properties:
        kibana_version:  { get_input: es_version }
        elasticsearch_password: { get_input: kibana_password }
      requirements:
        - host: server
        - dependency: elasticsearch
```

``` yaml tab="tosca custom type" hl_lines="22 27"
  tosca.nodes.indigo.Kibana:
    derived_from: tosca.nodes.SoftwareComponent
    properties:
      kibana_version:
        type: string
        required: false
        default: 7.4.1
      elasticsearch_url:
        type: string
        required: false
        default: "http://localhost:9200"
      elasticsearch_username:
        type: string
        required: false
        default: kibana
      elasticsearch_password:
        type: string
        required: false
        default: changeme
    artifacts:
      es_role:
        file: maricaantonacci.kibana,7.4.1
        type: tosca.artifacts.AnsibleGalaxy.role
    interfaces:
      Standard:
        configure:
          implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/k8s_add_es/artifacts/elk/kibana_install.yml
          inputs:
            kibana_version: { get_property: [ SELF, kibana_version ] }
            kibana_elasticsearch_url: { get_property: [ SELF, elasticsearch_url ] }
            kibana_elasticsearch_username: { get_property: [ SELF, elasticsearch_username ] }
            kibana_elasticsearch_password: { get_property: [ SELF, elasticsearch_password ] }
```

``` yaml tab="ansible artefact"
---
- hosts: localhost
  connection: local
  vars:
  roles:
    - role: maricaantonacci.kibana
```


    

