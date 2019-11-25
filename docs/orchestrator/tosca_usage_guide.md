### [◀](../README.md)

# TOSCA Usage Guide

## TOSCA Grammar

### Simplified topology template

``` yaml
topology_template:
  description: <template_description>
  inputs: <input_parameter_list>
  outputs: <output_parameter_list>
  node_templates: <node_template_list>
  relationship_templates: <relationship_template_list>
  policies:
    - <policy_definition_list>
```

#### Node template

A Node template is an instance of a specified Node Type and can provide customized properties, constraints or operations which override the defaults provided by its Node Type and its implementations.

``` yaml
<node_template_name>:
  type: <node_type_name>
  properties:
    <property_definitions>
  requirements:
    <requirement_definitions>
  capabilities:
    <capability_definitions>
  interfaces:
    <interface_definitions>
  artifacts:
    <artifacts_definitions>
```

- **interface_definitions**: represents the optional list of interface definitions for the Node Template that augment those provided by its declared Node Type.
- **artifact_definitions**: represents the optional list of artifact definitions for the Node Template that augment those provided by its declared Node Type.

### Node type

``` yaml
<node_type_name>:
  derived_from: <parent_node_type_name>
  description: <node_type_description>
  properties:
    <property_definitions>
  attributes:
    <attribute_definitions>
  requirements:
    - <requirement_definition_1>
    ...
    - <requirement_definition_n>
  capabilities:
    <capability_definitions>
  interfaces:
    <interface_definitions>
  artifacts:
    <artifact_definitions>
```

#### Example

** tosca.nodes.Root **

The TOSCA Root Node Type is the default type that all other TOSCA base Node Types derive from.

``` yaml
tosca.nodes.Root:
  description: The TOSCA Node Type all other TOSCA base Node Types derive from
  attributes:
    tosca_id:
      type: string
    tosca_name:
      type: string
    state:
      type: string
  capabilities:
    feature:
      type: tosca.capabilities.Node
  requirements:
    - dependency:
        capability: tosca.capabilities.Node
        node: tosca.nodes.Root
                 relationship: tosca.relationships.DependsOn
                 occurrences: [ 0, UNBOUNDED ]
  interfaces:
    Standard:
      type: tosca.interfaces.node.lifecycle.Standard
```

** tosca.nodes.Compute **

``` yaml
tosca.nodes.Compute:
  derived_from: tosca.nodes.Root
  attributes:
    private_address:
      type: string
    public_address:
      type: string
    networks:
      type: map
      entry_schema:
        type: tosca.datatypes.network.NetworkInfo
    ports:
      type: map
      entry_schema:
        type: tosca.datatypes.network.PortInfo
  requirements:
    - local_storage:
        capability: tosca.capabilities.Attachment
        node: tosca.nodes.BlockStorage
        relationship: tosca.relationships.AttachesTo
        occurrences: [0, UNBOUNDED] 
  capabilities:
    host:
      type: tosca.capabilities.Container
      valid_source_types: [tosca.nodes.SoftwareComponent]
    endpoint:
      type: tosca.capabilities.Endpoint.Admin
    os:
      type: tosca.capabilities.OperatingSystem
    scalable:
      type: tosca.capabilities.Scalable
    binding:
      type: tosca.capabilities.network.Bindable
```

### Capability Type

``` yaml
<capability_type_name>:
  derived_from: <parent_capability_type_name>
  description: <capability_description>
  properties:
    <property_definitions>
```

#### Example

** tosca.capabilities.Container ** 
``` yaml
tosca.capabilities.Container:
  derived_from: tosca.capabilities.Root
  properties:
    num_cpus:
      type: integer
      required: false
      constraints:
        - greater_or_equal: 1
    cpu_frequency:
      type: scalar-unit.frequency
      required: false
      constraints:
        - greater_or_equal: 0.1 GHz
    disk_size:
      type: scalar-unit.size
      required: false
      constraints:
        - greater_or_equal: 0 MB
    mem_size:
      type: scalar-unit.size
      required: false
      constraints:
        - greater_or_equal: 0 MB
```
### Relationship Type

``` yaml
<relationship_type_name>:
  derived_from: <parent_relationship_type_name>
  description: <relationship_description>
  properties:
    <property_definitions>
  attributes:
    <attribute_definitions>
  interfaces:
    <interface_definitions>
  valid_target_types: [ <entity_name_or_type_1>, ..., <entity_name_or_type_n> ]
```


The basic relationship types are: 

- **dependsOn**: abstract type and its sub types:
   - **hostedOn**:  a node is contained within another
   - **connectsTo**: a node has a connection configured to another


### Functions

#### get_input

The get_input function is used to retrieve the values of properties declared within the inputs section of a TOSCA Service Template.

``` yaml tab="Grammar"
get_input: <input_property_name>
```

``` yaml tab="example"
inputs:
  cpus:
    type: integer
  node_templates:
    my_server:
      type: tosca.nodes.Compute
      properties:
        num_cpus: { get_input: cpus }
```

#### get_property

The get_property function is used to retrieve property values between modelable entities defined in the same service template.
Use this function for inputs parameters.

``` yaml tab="Grammar"
get_property: [ <modelable_entity_name | SELF | SOURCE | TARGET | HOST>, [<capability_name>], <property_path> ]
```

``` yaml tab="example"
node_templates:
  mysql_database:
    type: tosca.nodes.Database
    properties:
      name: sql_database1
 

  wordpress:
    type: tosca.nodes.WebApplication.WordPress
    ...
    interfaces:
      Standard:
        configure:
          inputs:
            wp_db_name: { get_property: [ mysql_database, name ] }
```

#### get_attribute

The get_attribute function is used to retrieve the values of named attributes declared by the referenced node or relationship template name.

``` yaml tab="Grammar"
get_attribute: [ <modelable_entity_name | SELF | SOURCE | TARGET | HOST>, <attribute_name>]
```

#### concat

The concat function is used to concatenate two or more string values within a TOSCA service template.


``` yaml tab="Grammar"
concat: [ <string_value_expressions_*> ]
```

``` yaml tab="example"
outputs:
  description: Concatenate the URL for a server from other template values
  server_url:
  value: { concat: [ 'http://',
                     get_attribute: [ server, public_address ],
                     ':',
                     get_attribute: [ server, port ] ] }
```
