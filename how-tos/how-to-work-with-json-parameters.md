# How to work with JSON parameters
**Tested on:** OpenStack Queens

## Solution
JSON is one of the supported OpenStack Heat parameter data types. If you wish to exact a key's value from a json parameter, the desired value can be selected by using the `get_param` function with a list that references the key name.

For example, if the following json parameter is provided:

```yaml
parameters:
    coordinates:
        type: json
        default: { x_axis: 10, y_axis: 5 }
```

Then the ```x_axis``` value can be selected in the following way:

```yaml
value: { get_param: [ coordinates, x_axis ] }
```

## Full test template
_example.yaml:_
```yaml
heat_template_version: queens

parameters:
    coordinates:
        type: json
        default: { x_axis: 10, y_axis: 5 }

resources:
    x_coordinate:
        type: OS::Heat::Value
        properties:
            value: { get_param: [ coordinates, x_axis ] }

outputs:
    x_value:
        value: { get_attr: [ x_coordinate, value ] }
```
