--------

--------

# Using the Mock Devices<a name="iot-tg-gs-mockdevices"></a>

Two of the examples in this section, [Creating a Flow in the Cloud with Devices](iot-tg-gs-thing-sample-cloud.html) and [Creating a Flow in an AWS IoT Greengrass Group with Devices](iot-tg-gs-thing-sample.html), provide Python scripts that mock the behavior of real MQTT devices in a flow\. These scripts subscribe and publish to MQTT topics that AWS IoT Things Graph uses to mediate the steps in a flow\. This topic explains what these scripts do, and then describes how you can modify them to test your own flows when you don't have real devices\.

The cloud mock device scripts are in the [CloudMockDevices\.zip](samples/CloudMockDevices.zip) file\. The AWS IoT Greengrass mock device scripts are in the [MockDevices\.zip](samples/MockDevices.zip) file\.

## What the Mock Devices Do<a name="iot-tg-gs-mockdevices-what"></a>

In flows that include MQTT devices, AWS IoT Things Graph uses MQTT topics to mediate the communications between the devices\. In the example flows that include a motion sensor, a camera, and a screen, the motion sensor publishes a JSON object that indicates that it has detected motion\. Because the motion sensor mock device isn't real, it sends the message every 10 seconds\. The camera must both subscribe and publish to MQTT topics because it takes input and generates output\. The camera subscribes to a topic where AWS IoT Things Graph publishes the instruction to take a picture\. After the camera takes the picture, it publishes a JSON object containing the URI of the resulting image to another topic\. The screen subscribes to a topic where AWS IoT Things Graph publishes the image URI\. Because the screen mock device isn't real, it simply prints the image URI to standard output\.

The motion sensor mock device sends the following JSON to `Motion Sensor Thing Name/motion` every 10 seconds\.

```
      {"isMotionDetected": true}
```

When the motion sensor publishes this message, it tells AWS IoT Things Graph that the motion sensor's state has changed\. The `isMotionDetected` value and Boolean data type match the single field in the `MotionSensorState` entity, as defined in the [Aukru HCSR501 motion sensor](iot-tg-examples-motionsensor.html) definition\.

```
        type MotionSensorState @stateType(id: "urn:tdm:aws/examples:State:MotionSensorState") {
        isMotionDetected: Boolean @property(id: "urn:tdm:aws:property:Boolean")
      }
```

The capability implementation in the [Aukru HCSR501 motion sensor](iot-tg-examples-motionsensor.html) definition specifies the MQTT topic to which the motion sensor publishes the message\. You can find this topic inside the `Event` block\.

**Note**  
If the MQTT message responds to a device event, you'll find the relevant topic or topics in the `Event` block\. If it occurs as part of a device action, you'll find the relevant topics in the `Action` block\.

```
        MQTT {
            MotionSensorCapability(id: "urn:tdm:aws/examples:capability:MotionSensorCapability") {
              ...
                Event(name: "StateChanged") {
                    Subscribe(topic: "$macro(${systemRuntime.deviceId}/motion)") {
                            responsepayload(property: "urn:tdm:aws/examples:property:MotionSensorStateProperty")
                    }
                }
              }
            }
```

When the motion sensor's state changes, AWS IoT Things Graph receives the message on the topic specified in the `StateChanged` event: the device ID \(the name of the thing in the AWS IoT registry that is being used in the flow\) and `/motion`\. This is the topic to which the motion sensor publishes\.

 The camera mock device listens for capture commands that AWS IoT Things Graph publishes to `Camera Thing Name/capture`\. Because the flow doesn't specify any input values for the camera action, the JSON object published to this topic is empty\. The camera then publishes the following JSON to `Camera Thing Name/capture/finished`\. 

```
{"imageUri":"Image URI"}
```

The `imageUri` value and string data type match the single field in the `CameraState` entity as defined in the [Raspberry Pi camera definition](iot-tg-examples-rpicamera.html)\.

```
        type CameraState @stateType(id: "urn:tdm:aws/examples:state:CameraState") {
            lastClickedImage : Uri @property(id: "urn:tdm:aws:property:String")
        }
```

The capability implementation in the [Raspberry Pi camera](iot-tg-examples-rpicamera.html) definition specifies the MQTT topics to which the camera subscribes and publishes messages\. You can find them inside the `Publish` block of the `Action` block\.

```
        MQTT {
            CameraCapability(id: "urn:tdm:aws/examples:capability:CameraCapability") {
              ...
                Action(name: "capture") {
                    Publish {
                        Request(topic: "$macro(${systemRuntime.deviceId}/capture)") {
                            params
                        }
                        Response(topic: "$macro(${systemRuntime.deviceId}/capture/finished)") {
                            responsePayload(property: "urn:tdm:aws/examples:property:CameraStateProperty")
                        }
                    }
                }
              }
            }
```

When the value of the property of `lastClickedImage` in the device's state changes, AWS IoT Things Graph publishes an empty request to the `device ID (thing name)/capture` topic\. It then expects the camera to publish a response to the `device ID (thing name)/capture/finished` topic\.

After receiving the `imageUri`, AWS IoT Things Graph publishes a JSON object that is identical to the one that the camera sends to the `Screen Thing Name/display` topic\.

The capability implementation in the [Raspberry Pi screen](iot-tg-examples-rpiscreen.html) definition specifies the MQTT topic to which the screen subscribes\. You can find it inside the `Publish` block of the `Action` block\.

```
         MQTT {
            ScreenCapability(id: "urn:tdm:aws/examples:capability:ScreenCapability") {
              ...
                Action(name: "display") {
                    params {
                        param(name:"imageUrl" property:"urn:tdm:aws:property:String")
                    }
                    Publish {
                        Request(topic: "$macro(${systemRuntime.deviceId}/display)") {
                            params {
                                param(name: "imageUri", property: "urn:tdm:aws/examples:property:CameraStateProperty", value: "${imageUrl.value}")
                            }
                        }
                    }
                }
            }
        }
```

AWS IoT Things Graph gets the image URI from the camera, and then passes it as a parameter to the `Action` implementation\. The `Action` implementation tells AWS IoT Things Graph to publish the image URI to the `device ID (thing name)/display` topic\.

## Adapting the Mock Devices<a name="iot-tg-gs-mockdevices-using"></a>

The Python mock devices show how to use the AWS IoT Device SDK for Python and the AWS IoT Greengrass Core SDK to subscribe and post to MQTT topics in the cloud and in an AWS IoT Greengrass group\. They also demonstrate at a high level how MQTT devices interact with AWS IoT Things Graph when they perform steps in a flow\. When adapting the mock devices, you need to change only the MQTT topics and JSON payloads to match the topics, inputs, and outputs that you've specified in your own flows\. You can write your own mock devices using your preferred languages and SDKs\. 

When adapting the Python mock device scripts, look for the lines containing `myAWSIoTMQTTClient.subscribe` and `myAWSIoTMQTTClient.publish`\. For example, the `cloudcamera.py` script subscribes to the `thingName/capture` topic on line 129\.

```
myAWSIoTMQTTClient.subscribe(thingName + "/capture", 0, customCallback)
```

The first parameter of the `subscribe` method specifies the topic to which the mock device subscribes\. You can use this method to subscribe to any MQTT topic that you've specified in your flow\. You can create your own callback function if you want the mock device to do something other than print a message whenever it receives a message on the topic\.

The `cloudcamera.py` script publishes to the `thingName/capture/finished` topic on line 120\.

```
myAWSIoTMQTTClient.publish(thingName + "/capture/finished", messageJson, 0)
```

The first parameter of the `publish` method specifies the topic to which the mock device publishes\. You can use this method to publish to any MQTT topic that you've specified in your flow\. The second parameter specifies the JSON object that contains the message to publish\. You can change this value to match the input and output values that you've specified in your flow\.

The mock device scripts in [Creating a Flow in an AWS IoT Greengrass Group with Devices](iot-tg-gs-thing-sample.html) also demonstrate how to discover and connect to an AWS IoT Greengrass group so that devices in the group can interact with each other\. This is the necessary first step before devices start sending messages to AWS IoT Things Graph when it is running on an AWS IoT Greengrass core device\.