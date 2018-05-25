Home Automation
###############

:date: 2018-05-25
:tags: home-automation

MQTT
----
My own protocol, structure and convention on top of MQTT (as transport layer) follows closed <a href="https://github.com/homieiot/convention">HomieIoT</a>. This is done for two reasons: a) it's already developed, tested, and well-thought out, and b) actual homie nodes can be easily integrated.

* A device is a physical piece of hardware and can contain one or multiple nodes (sensors or actuators). A node can have multiple properties.
* The base topic is "home/".
* The topic for a device is "&lt;BASE_TOPIC&gt;/&lt;DEV_ID&gt;/" with DEV_ID usually being the room or room-function.
* Device attributes that MUST BE set even if the homie firmware isn't used are:
  - $name: friendly name
  - $state: at least "init", "ready", "alert"
  - $localip: IPv4 address of the device
  - $mac: MAC address of the device
  - $fw/name and $fw/version
  - $nodes: comma-separated list of nodes
  - $stats/uptime in seconds, $stats/interval in seconds, $stats/signal if ESP8266

* The topic for a node is "&lt;BASE_TOPIC&gt;/&lt;DEV_ID&gt;/&lt;NODE_ID&gt;/" with NODE_ID describing a subsystem of the node, e.g. "sensors".
* Node attributes MUST BE:
  - $name: friendly name
  - $type: string describing the type
  - $properties: comma-separated list of exposed sensor values or actuators

* The topic for a property is "&lt;BASE_TOPIC&gt;/&lt;DEV_ID&gt;/&lt;NODE_ID&gt;/&lt;PROPERTY_ID&gt;/" with PROPERTY_ID usually being the type of sensor, e.g. "temperature". The property value can be read from the property topic.
* Node attributes MUST BE:
  - $name: friendly name
  - $settable: writeable property or readonly
  - $unit: unit string, e.g. "°C", "W", "%", "#" (count), "" (empty)
  - $datatype: string describing the datatype: "integer", "float", "boolean", "string"
  - $properties: comma-separated list of exposed sensor values or actuators
  - set: if settable, write value based on datatype to set the property

Examples
--------
* Central room sensors device: home/livingroom_central/
  - $name = "Living Room Central"
  - $state = "ready"
  - $localip = "10.1.3.134", $mac = "EF:EF:AA:BB:CC:56"
  - $fw/name = "esp32-livingroom-central", $fw/version = "1.0.0"
  - $stats/uptime = 180, $stats/interval = 60, $stats/signal = 80
  - $nodes = "sensors,heatpump,corner-light":

1. sensors

  1. $name = "Living Room Sensors"
  2. $type = "sensors"
  3. $properties = "temperature,humidity":

    1. temperature: $name = "Living Room Temperature", $settable = "false", $unit = "°C", $datatype = "float"
    2. humidity: $name = "Living Room Humidity", $settable = "false", $unit = "%", $datatype = "integer"

2. heatpump

  1. $name = "Living Room Heatpump Control"
  2. $type = "ir"
  3. $properties = "heating":

    1. heating: $name = "Heatpump Heating Enabled", $settable = "true", $datatype = "boolean"

3. corner-light

  1. $name = "Living Room Corner Light"
  2. $type = "light"
  3. $properties = "powered":

    1. heating: $name = "Living Room Corner Light Powered", $settable = "true", $datatype = "boolean"
