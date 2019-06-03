--------

--------

# Creating Flow Configurations \(Cloud\)<a name="iot-tg-sysdeploy-depconfig-cloud"></a>

A [flow configuration](iot-tg-models-tdm-iot-sdc-deployconfig.html) specifies all of the components and properties required to deploy a system in a physical location\. The flow configuration consists of an AWS IoT Things Graph Data Model \(TDM\) deployment definition\. The attributes in the definition specify other required resources, such as the IAM role that AWS IoT Things Graph assumes when it executes the flow in the cloud\.

This topic describes the structure of a flow configuration and shows how to create one with the AWS IoT Things Graph console and the AWS CLI\.

## Creating a Flow Configuration in the AWS IoT Things Graph console<a name="iot-tg-sysdeploy-depconfig-console"></a>

These instructions are similar to the procedure for creating and deploying the flow in [Creating a Flow with Devices and a Service](iot-tg-gs-thingdev-sample.html)\. That topic contains complete instructions on setting up your environment, creating things, creating the flow \(and the system\), and associating things with devices\. Because this is a cloud deployment, instead of creating and configuring a AWS IoT Greengrass group, follow the instructions in [Prepare for Cloud Deployments](iot-tg-gs-environment-cloud.html) to create the IAM role that AWS IoT Things Graph assumes when it executes the flow\.

1. Create the flow configuration\.

   Select the menu icon at the upper left of the page, and then select **Flows** to return to the **Flows** page\. On the **Flows** page, select the box next to the flow that you just created, and then choose **Create flow configuration**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGCreateRekDeployment.png)

1. Name the flow configuration\.

   On the **Describe flow configuration** page, select your flow, and then enter a flow configuration name\. The flow configuration name can't contain spaces\. Choose **Cloud**, and then choose **Next**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGRekCloudDeploymentDetails.png)

1. Configure the target\.

   On the **Configure target page**, enter the ARN of the role that you created in Prepare for Cloud Deployments\. Choose **Next**\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGCloudDeploymentDetails2.png)

1. Select things for your deployment\.

   The **Map Things** page provides an interface for selecting the specific things that you'll include in your deployment\. The menus under each device in your deployment contain all of the things that you associated with the device\. Because you're getting started, the menus for each device on this page will include only one thing \(the thing that you've associated with each device\)\.

   On the **Map Things** page, under the **MotionSensor** device, select the motion sensor thing that you created earlier\. Select the camera and screen things for the **Camera** and **Screen** devices\. Choose **Next**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGRekSelectThings.png)

1. View the trigger\.

   On the **Define trigger** page, the GraphQL that defines the motion event trigger for the flow appears in the editor\. When the motion sensor detects motion, the `ThingsGraph.startFlow` function initiates the flow\. You don't need to edit this code\.

   Choose **Review**\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGRekDefineTrigger.png)

1. Review and create\.

   On the **Review and create** page, review the information you entered for your flow configuration\. Choose **Create**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGReviewDeployment.png)

1. Deploy\.

   When the **Flow configuration created** message appears, choose **Deploy now**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDeploymentCreated.png)

   After a successful deployment, the **Deployments** page displays **Deployed in target** in the **Status** column\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDeploySuccess.png)

## Creating a Flow Configuration in the AWS CLI<a name="iot-tg-sysdeploy-cloud-depconfig-cli"></a>

These instructions are similar to the procedure for creating and deploying the flow in [Creating a Flow with Devices and a Service](iot-tg-gs-thingdev-sample.html)\. That topic contains complete instructions on setting up your environment, creating things, creating the flow \(and the system\), and associating things with devices\. Because this is a cloud deployment, instead of creating and configuring a AWS IoT Greengrass group, follow the instructions in [Prepare for Cloud Deployments](iot-tg-gs-environment-cloud.html) to create the IAM role that AWS IoT Things Graph assumes when it executes the flow\.

### Defining the Flow Configuration<a name="iot-tg-sysdeploy-depconfig-cli-defdoc"></a>

The following AWS IoT Things Graph Data Model \(TDM\) shows the underlying definition of the flow configuration created in [Creating a Flow with Devices and a Service](iot-tg-gs-thingdev-sample.html)\. Replace the *REGION* and *ACCOUNT ID* placeholders with your AWS Region and account ID\.

```
query Lobby @deployment(id: "urn:tdm:REGION/ACCOUNT ID/default:Deployment:Lobby" systemId: "urn:tdm:REGION/ACCOUNT ID/default:System:RekognitionFlowSystem") {
    motionSensor(deviceId: "MotionSensor1")
    screen(deviceId: "Screen1")
    cameraRkgnExample(deviceId: "Camera1")
    
    triggers {
        MotionEventTrigger(description: 'a trigger') {
            condition(expr: "devices[name == 'motionSensor'].events[name == 'StateChanged'].lastEvent.value") 
            action(expr: "ThingsGraph.startFlow('RekognitionFlow', bindings[name == 'cameraRkgnExample'].deviceId, bindings[name == 'screen'].deviceId)")
        }
    }
}
```

In this deployment definition:
+ You create a deployment definition as a GraphQL `query`\.
+ The `@deployment` declaration tells AWS IoT Things Graph to create a [deployment](iot-tg-models-tdm-iot-sdc-deployconfig.html) definition whose identifier is the [TDM URN](https://docs.aws.amazon.com/thingsgraph/latest/ug/iot-tg-models-tdm-urnscheme.html) inside the parentheses\. 
+ The next value inside the parentheses is the URN of the system that contains the workflow \(also called a flow\) or flows that deploy with this deployment\.
+ The content inside the braces begins by assigning the names of the devices or device models in the flows \(as specified in the system definition\) to things that are registered in your AWS IoT registry\. Before you make this assignment in the TDM, use the [AssociateEntityToThing](https://docs.aws.amazon.com/thingsgraph/latest/APIReference/API_AssociateEntityToThing.html) API to associate each device with each thing you're using in the deployment\. The `deviceId` values are the names of the things in your registry\.
+ The `triggers` block contains one or more [triggers](iot-tg-models-tdm-iot-trigger.html)\. The `MotionEventTrigger` in this example consists of a `condition` that triggers the flow and the `action` that starts when the trigger condition is met\.
+ The condition uses two path [expressions](iot-tg-models-tdm-expressions.html) to identify a device that is used in a flow and one of that device's events, with a predicate expression that determines whether the device has detected motion\. If the `lastEvent` value sent by the `motionSensor` device is true, a non\-empty string, or a numeric value other than zero \(0\), the `devices[name == 'motionSensor'].events[name == 'StateChanged'].lastEvent.value` expression evaluates to true\. This signifies that a motion detected event has occurred\. If you want the expression to evaluate to true every time the event is fired, use `devices[name == 'motionSensor'].events[name == 'StateChanged'].lastEvent`\.
+ The `action` uses the `ThingsGraph.startFlow` function, which initiates the specified flow\. The flow name matches the name of the flow in the system definition\. The `bindings` path expressions specify the names of the things that are used in the flow\.

Now that you have a complete deployment definition, you can create the flow configuration by using the AWS CLI\.

The values that follow the flow name inside the action can be any valid flow parameters\. The following example triggers show some of the values and path expressions that you can use inside the action block of a trigger\.

```
DataEmitterTrigger01(description: "trigger on integer events") {
        condition(expr: "devices[name == 'dataEmitter'].events[name == 'IntegerEvent'].lastEvent.val > 10")
        action(expr: "ThingsGraph.startFlow('exampleFlow1', devices[name == 'dataEmitter'].events[name == 'IntegerEvent'].lastEvent.val, False, 1.1, ((String)devices[name == 'dataEmitter'].events[name == 'IntegerEvent'].lastEvent.val).charAt(0))")
      }

DataEmitterTrigger02(description: "trigger on json events") {
        condition(expr: "devices[name == 'dataEmitter'].events[name == 'JsonEvent'].lastEvent")
        action(expr: "ThingsGraph.startFlow('exampleFlow2', -100, True, -1.0000000001, devices[name == 'dataEmitter'].events[name == 'JsonEvent'].lastEvent)")
      }
```

For more information about triggers, see [Trigger](iot-tg-models-tdm-iot-trigger.html)\.

### Creating the Flow Configuration<a name="iot-tg-sysdeploy-depconfig-cli-create"></a>

The following command shows how to create a flow configuration by using the AWS CLI\.

```
aws iotthingsgraph create-system-instance --definition language=GRAPHQL,text="TDM Deployment Definition" \
--target CLOUD --flow-actions-role-arn IAM Role \
--metrics-configuration true/false
```
+ `target` parameter – Specifies the target type of the deployment\. For cloud deployments, specify `CLOUD`\.
+ `flow-actions-role-arn` parameter – Specifies the role that AWS IoT Things Graph assumes when it executes the flow in the cloud\. For instructions on how to create this role, see [Prepare for Cloud Deployments](iot-tg-gs-environment-cloud.html)\.

When the operation completes, the AWS CLI returns the following deployment summary \(as a JSON object\)\.

```
{
        "summary": {
            "status": "PENDING_DEPLOYMENT",
            "greengrassGroupName": "AWS IoT Greengrass Group Name",
            "target": "GREENGRASS",
            "arn": "arn:aws:iotthingsgraph:REGION:ACCOUNT ID:default#Deployment#Lobby",
            "updatedAt": 1547245009.256,
            "id": "urn:tdm:REGION/ACCOUNT ID/default:Deployment:Lobby",
            "createdAt": 1547245009.256
        }
}
```

In this deployment summary:
+ The `status` value in the `summary` object is `PENDING_DEPLOYMENT` when a flow configuration is created\.
+ The `id` value in the `summary` block is the TDM URN of the flow configuration\.

 The following command uses the `id` value to deploy the flow configuration to the target\.

```
aws iotthingsgraph deploy-system-instance --id Flow Configuration Id
```

For more information about creating a flow configuration programmatically, see [CreateSystemInstance](https://docs.aws.amazon.com/thingsgraph/latest/APIReference/API_CreateSystemInstance.html) in the [AWS IoT Things Graph API Reference](https://docs.aws.amazon.com/thingsgraph/latest/APIReference/)\.