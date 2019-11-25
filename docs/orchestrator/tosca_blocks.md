### [◀](../README.md)

## TOSCA template

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
