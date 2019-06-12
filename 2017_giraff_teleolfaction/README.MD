#Launchfiles

* Giraf: launch "giraff.launch" on giraff
* Remote_control: launch "remote.launch" on remote control station

Both launchfiles include all the required sub-launchfiles


#Notes

## HiveMQ server

* IP: 150.214.109.137
* Port:
    * 8000: web application
    * 8002: robot

**To interchange messages over HiveMQ**: launch the remote_mqtt_bridge node (should automatically happen with the included launchfiles) and the mapir-iot-api

* MQTT_bridge:
     * Subscribes to rostopic /ros2mqtt, type diagnostic_msgs::KeyValue, which contains a key (string) and a value (string).
        * On received msg on /ros2mqtt, it publishes a message to MQTT over mosquitto, under the topic MQTT_topicName/KeyValue.
            * MQTT_topicName can be ocnfigured in the launchfile to avoid robot colission, in my case it is **andresgongora_teleolfaction/**
            * KeyValue is the key string contained in /ros2mqtt

    * It subscribes to the MQTT topics specified in the launchfile, such as `NavigationCommand` (specified in launchfile) as follows: `MQTT_topicName/NavigationCommand`.
        * On received message over MQTT, it converts the message to a key-value string pair (diagnostic_msgs::KeyValue) and publishes to `/mqtt2ros`, which will be processed by `mapir-iot-api`.

* MAPIR_iot_api:
    * Subscribe to `/mqtt2ros`
        * On received message check key and process value accordingly. Usually by generating a message of the apropriate type and publishing it under a launchfile-specified ros topic name.
    * Subscribe to hard-coded topics (name can be specified in launchfile, but not their type) and converts them to a key-value pair which can be published on `/ros2mqtt` to start the loop again. 

**Send data from ROS->MQTT**

`ROS>>> ---[ros-topic]---> mapir-iot_api ---[ros2mqtt]---> MQTTBRIGDE ---[MQTT_topicName/KeyValue]--->>> MQTT`

**Receive data from MQTT->ROS**

`MQTT >>>---[MQTT_topicName/KeyValue]---> MQTTBRIGDE ---[mqtt2ros]---> mapir-iot_api ---[ros-topic]--->>> ROS`


## Topics

### ROS

* **/olfaction/mapir_enose**:  
* **/olfaction/normalized_gas_sensor**:`olfaction_msgs::gas_sensor` a gas measurement in the range [0,1]. All other values (e.g. manufacturer and technology) are set to 
* **/olfaction/anemometer**: `olfaction_msgs::gas_sensor`

### MQTT

* **GasSensor**: a single olfaction_msgs::gas_sensor converted to key-value, space separated.
* **Anemometer**: a single olfaction_msgs::anemometer converted to key-value, space separated.


# Special considerations

* simple_enose_normalizer publishes the logarithm of the actual normal value to increase resolution. People work in logarithmic scale, not linear
* If the e-nose does not work: `chmod 777 /dev/tty/USB0`

