# Models
In this section, we'll be describing the models that are part of the component specification.

Here we split by the major models: Events, Attributes, and the supported Configurations.

Everything is specified by using YAML files, but JSON could also be easily supported.

## Events
They represent an event that will be sent to analytics providers, they have the following properties:

- Name: Name that will be sent to the analytics provider
- Type:
  - _default_: Represents a regular event, an action that was executed
  - _screen_: Represents a view, like a user just opened and viewed something in the application

Below is a quick example of how an event is defined:
```yaml
name: "Shop - Start Buy"
type:
  name: "default"

scopes:
  - name: "screen"
    attributes:
      - name: "name"
        type: "string"
        value: "PDP"
        required: true

  - name: "product"
    attributes:
      - name: "articleid"
        type: "string"
        required: true
```

### Scopes and attributes
Each event needs at least one scope, they are used to group similar attributes to put some context around them. Later the scope name + attribute name will be used to generate final names that will be sent to the provider.

For every scope, we have a name and an attributes list as their properties. Retrieving from the example, here we have 2 scopes (screen and product): 
```yaml
scopes:
  - name: "screen"
    attributes:
      - name: "name"
        type: "string"
        value: "PDP"
        required: true

  - name: "product"
    attributes:
      - name: "articleid"
        type: "string"
        required: true
```

For the attributes, they represent the parameters that the developer will send inputs. They have the following properties:
- Name: will be used to generate the final name that will be sent to the analytics provider, the scope name will be pre-pended to it along with a separator that is configured in the configuration file
- Type: Needs to be of one of the supported types, a few are supported and they are going to be used to enforce strong typing and more safety when generating the code. You can check more about this in the next section
- Required: Either if the attribute is mandatory or not, if it's not mandatory, it's going to be an optional parameter
- Value: If there's a value already pre-set, this attribute won't be mapped as a parameter, but it will always be sent to the provider with the value set here. You should use it if in your use case you always have to send the same value

### Supported types
Below are the supported types and a description of them:
- string: For regular strings
- float: For regular floating-point numbers
- integer: For integer numbers, negative or non-negative
- boolean: For boolean types, will be converted to true/false
- enum: Used for custom-defined enumerators (section [Enumerators](#Enumerators))

Chronological types:
- datetime: Represents a date + time, for example `day/month/year hour:minutes:seconds`
- time: Represents a time only
- date: Represents just the date

For all chronological types, the specification supports pre-defined formats inside the configuration file (we'll describe it in a future section)

You can also use `value: "now"` for chronological types, in case you do that, your attribute will automatically have the current time/datetime set, useful for logging purposes.

Collection types:
- map: A map where the key and value are a string, used to send custom parameters to any event
- list: A list of one of the supported types above (except map), you need to specify the type of the list using the character /. For example list/string

Since in the end, we'll be sending collection values as a string, there must be a delimiter string set inside the configuration file, this string will be used to join all elements of the list.

### Enumerators
To provide more type safety we also support the definition of custom enumerators, these will be converted to enumerators in the code and you can refer to them directly in the code.

All enumerators should be defined in their files and follow the structure below:

- Name: The name that will be used as the enumerator class name in the code, also you will refer to this in your code
- Values
  - Name: The name of the value that will be used in the code (preferably all uppercased)
  - value: The value (needs to be a string) that will be used to send to the analytics provider

Example:
```yaml
name: "ShopTab"
values:
  - name: "NOW"
    value: "Now"

  - name: "SOON"
    value: "Soon"
```

To have an attribute that uses this enumerator as its type you just have to use the following type:
```yaml
attributes:
    - name: "destination"
    type: "enum/ShopTab"
```

Below is an example of how this enumerator is generated in Kotlin:
```kotlin
public enum class ShopTab(
    public val `value`: String
) {
    NOW("Now"),
    SOON("Soon")
}
```

## Configurations
All configurations are set inside a file called configurations.yml at the root of the project.

You can see below a description of each supported setting in the file:
- name: The name of this package
- schema: The version of the analytics specification that this package uses
- datetime_format: ISO 8601 specification of the format that will be used to convert attributes of the type datetime
- date_format: ISO 8601 specification of the format that will be used to convert attributes of the type date
- time_format: ISO 8601 specification of the format that will be used to convert attributes of the type time
- scope_delimiter: A Delimiter used to merge the scope name plus the attribute name to create a key that will be sent to the analytics provider
- list_delimiter: A Delimiter used to merge multiple values for attributes of the type list
- default_event_attributes: Here you can specify scopes and attributes that will be added to all events. You should use the same format you use inside events.

Example:
```yaml
name: "MY app events"
schema: 1.0.0
datetime_format: "MM/dd/yy, HH:mm:ss"
date_format: "MM/dd/yy"
time_format: "HH:mm:ss"
scope_delimiter: ":"
list_delimiter: "|"
default_event_attributes:
  scopes:
    - name: "event"
      attributes:
        - name: "source"
          type: "string"
          value: "APP"
          required: true
```

With this configuration, all events will have their attributes names set as scope:attribute and each event will have an additional attribute called event:source with the pre-defined value set as "APP".

## File structure
Below you can see how is the file tree along with the folders and files for an analytics specification project:
- configurations.yml: Configurations object
- /enums
  - enum_with_spaces.yml (Use _ to represent spaces, each enumerator is a YML file)
- /events
  - /domain - Each screen could represent a domain or you can use your criteria to group events, but it's important to group them under folders.
    - event.yml