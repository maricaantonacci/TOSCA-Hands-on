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

### Capability Type

``` yaml
<capability_type_name>:
  derived_from: <parent_capability_type_name>
  description: <capability_description>
  properties:
    <property_definitions>
```

#### Example

### Capability definition

``` yaml
# Simple definition is as follows:
<capability_defn_name>: <capability_type>

# The full definition is as follows:
<capability_defn_name>:
  type: <capability_type>
  description: <capability_defn_description>
  properties:
    <property_values>
  attributes:
    <attribute_values>
  occurrences: <occurrences>
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

#### Example


