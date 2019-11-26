### [◀](../README.md)

## Installing Elasticsearch and Kibana with ansible

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

## Modeling the topology with TOSCA 

### Elasticsearch SoftwareComponent node

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
          implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/k8s/artifacts/elk/elasticsearch_install.yml
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

### Kibana SoftwareComponent node

``` yaml tab="tosca template excerpt" hl_lines="2 8"
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
          implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/k8s/artifacts/elk/kibana_install.yml
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

Now let's build the full topology template, adding the server node(s). 

_Don't forget to request port 5601 for accessing kibana dashboard!_



#### 1. All-in-One installation

``` yaml tab=" "
Click on the "TOSCA template" tab to see a possible solution.
```

``` yaml tab="TOSCA template"
tosca_definitions_version: tosca_simple_yaml_1_0

imports:
  - indigo_custom_types: https://raw.githubusercontent.com/maricaantonacci/tosca-types/k8s/custom_types.yaml

description: >
  Start Elasticsearch + Kibana on a Virtual Machine

topology_template:

  inputs:
    num_cpus:
      type: integer
      description: Number of virtual cpus for the VM
      default: 2
      constraints:
      - valid_values: [ 2, 4 ]
    mem_size:
      type: scalar-unit.size
      description: Amount of memory for the VM
      default: 4 GB
      constraints:
      - valid_values: [ 4 GB, 8 GB ]

    es_version:
      type: string
      default: 7.4.1
      description: Elasticsearch version

    es_bind_address:
      type: string
      default: 0.0.0.0
      description: Bind address for Elasticsearch service

    es_password:
      type: string
      required: true
      description: Password for user elastic

    kibana_password:
      type: string
      required: true
      description: Password for kibana system user

  node_templates:

    elasticsearch:
      type: tosca.nodes.indigo.Elasticsearch
      properties:
        es_version:  { get_input: es_version }
        bind_address: { get_input: es_bind_address }
        elastic_password: { get_input: es_password }
        kibana_system_password: { get_input: kibana_password }
      requirements:
        - host: kibana_es_server

    kibana:
      type: tosca.nodes.indigo.Kibana
      properties:
        kibana_version:  { get_input: es_version }
        elasticsearch_password: { get_input: kibana_password }
      requirements:
        - host: kibana_es_server
        - esearch_endpoint: elasticsearch

    kibana_es_server:
      type: tosca.nodes.indigo.Compute
      capabilities:
        endpoint:
          properties:
            network_name: PUBLIC
            ports:
              kibana:
                protocol: tcp
                source: 5601
        host:
          properties:
            num_cpus: { get_input: num_cpus }
            mem_size: { get_input: mem_size }
        os:
          properties:
            distribution: ubuntu
            type: linux
            version: 16.04

  outputs:
    kibana_endpoint:
      value: { concat: [ 'http://', get_attribute: [ kibana_es_server, public_address, 0 ], ':5601' ] }
    node_ip:
      value: { get_attribute: [ kibana_es_server, public_address, 0 ] }
    node_creds:
      value: { get_attribute: [ kibana_es_server, endpoint, credential, 0 ] }
```

#### 2. Installation on separate servers

``` yaml tab=" "
Click on the "TOSCA template" tab to see a possible solution.
```

``` yaml tab="TOSCA template"
tosca_definitions_version: tosca_simple_yaml_1_0

imports:
  - indigo_custom_types: https://raw.githubusercontent.com/maricaantonacci/tosca-types/k8s/custom_types.yaml

description: >
  Start Elasticsearch + Kibana on separate Virtual Machines

topology_template:

  inputs:
    num_cpus:
      type: integer
      description: Number of virtual cpus for the VM
      default: 2
      constraints:
      - valid_values: [ 2, 4 ]
    mem_size:
      type: scalar-unit.size
      description: Amount of memory for the VM
      default: 4 GB
      constraints:
      - valid_values: [ 4 GB, 8 GB ]

    es_version:
      type: string
      default: 7.4.1
      description: Elasticsearch version

    es_bind_address:
      type: string
      default: 0.0.0.0
      description: Bind address for Elasticsearch service

    es_password:
      type: string
      required: true
      description: Password for user elastic

    kibana_password:
      type: string
      required: true
      description: Password for kibana system user

  node_templates:

    elasticsearch:
      type: tosca.nodes.indigo.Elasticsearch
      properties:
        es_version:  { get_input: es_version }
        bind_address: { get_input: es_bind_address }
        elastic_password: { get_input: es_password }
        kibana_system_password: { get_input: kibana_password }
      requirements:
        - host: es_server

    kibana:
      type: tosca.nodes.indigo.Kibana
      properties:
        kibana_version:  { get_input: es_version }
        elasticsearch_password: { get_input: kibana_password }
        elasticsearch_url: { concat: [ 'http://', { get_attribute: [ es_server, private_address, 0 ] }, ":9200" ] }
      requirements:
        - host: kibana_server
        - esearch_endpoint: elasticsearch

    es_server:
      type: tosca.nodes.indigo.Compute
      capabilities:
        host:
          properties:
            num_cpus: { get_input: num_cpus }
            mem_size: { get_input: mem_size }
        os:
          properties:
            distribution: ubuntu
            type: linux
            version: 16.04

    kibana_server:
      type: tosca.nodes.indigo.Compute
      capabilities:
        endpoint:
          properties:
            network_name: PUBLIC
            ports:
              kibana:
                protocol: tcp
                source: 5601
        host:
          properties:
            num_cpus: { get_input: num_cpus }
            mem_size: { get_input: mem_size }
        os:
          properties:
            distribution: ubuntu
            type: linux
            version: 16.04

  outputs:
    kibana_endpoint:
      value: { concat: [ 'http://', get_attribute: [ kibana_server, public_address, 0 ], ':5601' ] }
    kibana_node_ip:
      value: { get_attribute: [ kibana_server, public_address, 0 ] }
    kibana_node_creds:
      value: { get_attribute: [ kibana_server, endpoint, credential, 0 ] }
```


### Run the deployment 

Choose one of the two topologies and submit the template to the Orchestrator:

``` bash
orchent depcreate template.yml '{ "es_password": "****", "kibana_password": "****" }'
```    

Monitor the status of the deployment:

``` bash
orchent depshow <dep UUID>
```

You could also connect to the dashboard to follow the deployment log:

`dashboard URL: https://dodas-paas.cloud.ba.infn.it`

What do you see in the log?

