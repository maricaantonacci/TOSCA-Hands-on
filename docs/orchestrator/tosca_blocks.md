### [◀](../README.md)


Let's build our first template describing a compute node.
We will use the indigo custom type **tosca.nodes.indigo.Compute** reported below.
Look at the full definition [here](https://github.com/indigo-dc/tosca-types/blob/master/custom_types.yaml).

Before reading the example tab, try to write down the node template respecting the node type definition. Then compare it with the example.


``` yaml tab="tosca indigo.Compute type"
  tosca.nodes.indigo.Compute:
    derived_from: tosca.nodes.indigo.MonitoredCompute
    attributes:
      private_address:
        type: list
        entry_schema:
          type: string
      public_address:
        type: list
        entry_schema:
          type: string
      ctxt_log:
        type: string
    capabilities:
      scalable:
        type: tosca.capabilities.indigo.Scalable
      os:
         type: tosca.capabilities.indigo.OperatingSystem
      endpoint:
        type: tosca.capabilities.indigo.Endpoint
      host:
        type: tosca.capabilities.indigo.Container
        valid_source_types: [tosca.nodes.SoftwareComponent]"
```

``` yaml tab="TOSCA node template (example)"
    server_node:
      type: tosca.nodes.indigo.Compute
      capabilities:
        scalable:
          properties:
            count: 1
        endpoint:
          properties:
            network_name: PUBLIC
            ports:
              http:
                protocol: tcp
                source: 80
        host:
          properties:
            num_cpus: 1
            mem_size: 2
        os:
          properties:
            distribution: ubuntu
            type: linux
            version: 16.04
```

!!! info 
    Pay attention at the [endpoint](https://github.com/indigo-dc/tosca-types/blob/master/custom_types.yaml#L149) capability that specifies the network configuration and the exposed ports.

Now let try to add the `inputs` section in order to allow the customization of the deployment. For example, we could provide input values for the node resources (cpu, ram).

!!! attention
    Don't forget:

      - the tag `tosca_definitions_version: tosca_simple_yaml_1_0` that must appear at the beginning of the template 

      - the `imports` tag:
        ``` yaml
        imports:
          - indigo_custom_types: https://raw.githubusercontent.com/indigo-dc/tosca-types/k8s/custom_types.yaml
        ```


``` yaml tab=" "
Click on the "TOSCA template" tab to see a possible solution.
```

``` yaml tab="TOSCA template"
tosca_definitions_version: tosca_simple_yaml_1_0

imports:
  - indigo_custom_types: https://raw.githubusercontent.com/indigo-dc/tosca-types/k8s/custom_types.yaml

description: >
  Launch a compute node getting the IP and SSH credentials to access via ssh

topology_template:

  inputs:        
    num_cpus:
      type: integer
      description: Number of virtual cpus for the VM
      default: 1
      constraints:
      - valid_values: [ 1, 2, 4 ]
    mem_size:
      type: scalar-unit.size
      description: Amount of memory for the VM
      default: 2 GB
      constraints:
      - valid_values: [ 2 GB, 4 GB ]  

    num_instances:
      type: integer
      description: Number of VMs to be spawned
      default: 1      

    os_distribution: 
      type: string
      default: ubuntu
 

  node_templates:

    simple_node:
      type: tosca.nodes.indigo.Compute
      capabilities:
        endpoint:
          properties:
            network_name: PUBLIC
        scalable:
          properties:
            count: { get_input: num_instances }
        host:
          properties:
            num_cpus: { get_input: num_cpus }
            mem_size: { get_input: mem_size }
        os:
          properties:
            distribution: { get_input: os_distribution }
            type: linux

  outputs:
    node_ip:
      value: { get_attribute: [ simple_node, public_address, 0 ] }
    node_creds:
      value: { get_attribute: [ simple_node, endpoint, credential, 0 ] }
```
