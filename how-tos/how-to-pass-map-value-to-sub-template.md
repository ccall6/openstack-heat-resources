# How to pass a map/dict value to a Heat sub-template
**Tested on:** OpenStack Queens

## Solution
OpenStack Heat allows custom resources to be instantiated within a Heat stack by referencing the sub-template's file name as the resource type:

```yaml
    example_sub:
        type: example.yaml
```

(Alternatively, you can define it as a resource type in the resource_registry within the environment file.)

Data values can be passed to these sub-templates by setting property values in the resource within the parent template. These resource properties will then pass their values to parameters of the same name defined within the sub-template. These sub-template parameters must have a data type specified, and when passing a map/dict value to a sub-template, the correct data type to use is `json`.

As an example, suppose a calling template wanted to pass the addresses attribute of a OS::Nova::Server resource (a map value) to a sub-template defined in the example.yaml template:

```yaml
    example_sub:
        type: example.yaml
        properties:
            vm_addresses: { get_attr: [ vm, addresses ] }
```

This would be matched within the sub-template through a json parameter of the same name:

```yaml
parameters:
    vm_addresses:
        type: json
```

## Full test templates
_parent.yaml:_
```yaml
heat_template_version: queens

resources:
    vm:
        type: OS::Nova::Server
        properties:
            flavor: "Tiny"
            image: "Cirros"
            name: "Cirros VM"
            networks:
            - port: { get_resource: vmi }

    ipam:
        type: OS::ContrailV2::NetworkIpam
        properties:
            name: "Cirros IPAM"

    vn:
        type: OS::ContrailV2::VirtualNetwork
        properties:
            name: "Cirros_VN"
            network_ipam_refs: [ { get_resource: ipam } ]
            network_ipam_refs_data:
            - network_ipam_refs_data_ipam_subnets:
                - network_ipam_refs_data_ipam_subnets_addr_from_start: true
                  network_ipam_refs_data_ipam_subnets_subnet: { network_ipam_refs_data_ipam_subnets_subnet_ip_prefix: "10.0.0.0",
                    network_ipam_refs_data_ipam_subnets_subnet_ip_prefix_len: "24"}

    vmi:
        type: OS::ContrailV2::VirtualMachineInterface
        properties:
            name: "Cirros VMI"
            virtual_network_refs: [ { get_resource: vn } ]

    instance_ip:
        type: OS::ContrailV2::InstanceIp
        properties:
            name: "Cirros Instance IP"
            virtual_machine_interface_refs: [ { get_resource: vmi } ]
            virtual_network_refs: [ { get_resource: vn } ]

    example_sub:
        type: example.yaml
        properties:
            vm_addresses: { get_attr: [ vm, addresses ] }

outputs:
    example_sub:
        value: { get_attr: [ example_sub, heat_value ] }
```

---

_example.yaml:_
```yaml
heat_template_version: queens

parameters:
    vm_addresses:
        type: json

resources:
    example_resource:
        type: OS::Heat::Value
        properties:
            value: { get_param: vm_addresses }

outputs:
    heat_value:
        value: { get_attr: [ example_resource, value ] }
```
