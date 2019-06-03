--------

--------

# Creating and Deploying Flows \(Cloud\)<a name="iot-tg-workflows-gs-cloud"></a>

After you model a flow and upload all of the entities that it contains, you create and deploy the flow\.

**Prerequisites**
+ An [AWS account](http://aws.amazon.com)
+ An IAM role that is configured according to the instructions in [Prepare for Cloud Deployments](iot-tg-gs-environment-cloud.html)
+ Completion of the steps in [Creating and Uploading Models](iot-tg-models-gs.html)

We provide procedures for creating and deploying flows using either the AWS CLI or the AWS IoT Things Graph console\. Both the AWS CLI and the console use the two devices that you created in [Creating and Uploading Models](iot-tg-models-gs.html)\.

## Create and Deploy a Flow \(CLI\)<a name="iot-tg-workflows-cli"></a>

The remaining Things Data Model \(TDM\) entities to create are the [Workflow](iot-tg-models-tdm-iot-workflow.html), [System](iot-tg-models-tdm-iot-system.html), and [Flow Configuration](iot-tg-models-tdm-iot-sdc-deployconfig.html)\. 

1. **Define the flow\.**

   The following GraphQL contains a definition of a flow that sends a message from a barcode reader to another device\.

   ```
   {
     query
     Example01($device1Id: String, $device2Id: String) @workflowType(id: "urn:tdm:REGION/ACCOUNT ID/default:Workflow:Example01") {
       variables {
         barcode @property(id: "urn:tdm:REGION/ACCOUNT ID/default:property:Barcode")
       }
       steps {
         step(name: "DeviceA", outEvent: ["step1_done"]) {
           DeviceActivity(deviceModel: "urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceA", deviceId: "${device1Id}", out: "barcode") @device(id: "urn:tdm:REGION/ACCOUNT ID/default:device:DeviceA") {
             readBarcode
           }
         }
         step(name: "DeviceB", inEvent: ["step1_done"]) {
           DeviceActivity(deviceModel: "urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceB", deviceId: "${device2Id}") @device(id: "urn:tdm:REGION/ACCOUNT ID/default:device:DeviceB") {
             doSomething(input: "${barcode}")
           }
         }
       }
     }
   }
   ```

1. **Create the flow\.**

   After you model the flow, you create it by using the `CreateWorkflowTemplate` API\. This API consumes a JSON object that contains two parameter values: `language` and `text`\. 

   Currently, the only supported value for `language` is GraphQL\. The value of `text` is a set of TDM definitions implemented in GraphQL\. 

   The resulting JSON looks like the following example\.

   ```
   {
       "language": "GRAPHQL",
       "text": "{query Example01($device1Id: String, $device2Id: String) @workflowType(id: 'urn:tdm:REGION/ACCOUNT ID/default:Workflow:Example01') {variables {barcode @property(id: 'urn:tdm:REGION/ACCOUNT ID/default:property:Barcode')} steps {step(name: 'DeviceA', outEvent: ['step1_done']) {DeviceActivity(deviceModel: 'urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceA', deviceId: '${device1Id}', out: 'barcode') {readBarcode}} step(name: 'DeviceB', inEvent: ['step1_done']) {DeviceActivity(deviceModel: 'urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceB', deviceId: '${device2Id}') {doSomething(input: '${barcode}')}}}}}"
   }
   ```

   After you construct the JSON payload that contains the GraphQL definition of the flow, you create the flow in your namespace by using the following CLI command\. 

   ```
    aws iotthingsgraph create-flow-template --definition file://flow file name  
   ```

1. **Define the system\.**

   In this step, you define a system that contains the flow you just created, and the devices that you modeled and associated with two things in your registry \(as described in [Getting Started with Models](iot-tg-models-gs.html)\)\. 

   The following GraphQL contains a system that includes your flow and those two things\. The URNs in the example are the unique identifiers of the device models that are associated with the things you created in [Getting Started with Models](iot-tg-models-gs.html)\.

   ```
   {
     type
     securitySystem @systemType(id: 'urn:tdm:REGION/ACCOUNT ID/default:system:Example01') {
       device1: Thing @thing(id: 'urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceA')
       device2: Thing @thing(id: 'urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceB')
       example01Flow: Workflow @workflow(id: 'urn:tdm:REGION/ACCOUNT ID/default:Workflow:Example01')
     }
   }
   ```

1. **Create the system\.**

   After you define the system, you create it by using the `CreateSystemTemplate` API\. This API consumes a JSON object that contains two parameter values: `language` and `text`\. 

   Currently, the only supported value for `language` is GraphQL\. The value of `text` parameter is a set of TDM definitions implemented in GraphQL\. 

   The resulting JSON looks like the following example\.

   ```
   {
       "language": "GRAPHQL",
       "text": "{type securitySystem @systemType(id: 'urn:tdm:REGION/ACCOUNT ID/default:system:Example01') {device1: Thing @thing(id: 'urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceA') device2: Thing @thing(id: 'urn:tdm:REGION/ACCOUNT ID/default:deviceModel:DeviceB') example01Flow: Workflow @workflow(id: 'urn:tdm:REGION/ACCOUNT ID/default:Workflow:Example01')}}"
   }
   ```

   After you construct the JSON payload that contains the GraphQL definition of the flow, you create the flow in your namespace by using the following CLI command\. 

   ```
    aws iotthingsgraph create-system-template --definition file://system file name
   ```

1. **Define the flow configuration\.**

   Systems are deployed within flow configurations\. 

   The following GraphQL defines a flow configuration for the system that you just created\. It passes a trigger to the flow that starts the flow every 10 seconds\.

   ```
   {
     query
     Example01 @deployment(id: 'urn:tdm:REGION/ACCOUNT ID/default:deployment:Example01', systemId: 'urn:tdm:REGION/ACCOUNT ID/default:system:Example01') {
       device1(deviceId: 'DeviceA')
       device2(deviceId: 'DeviceB')
       triggers {
         TenSecondTrigger(description: 'a trigger') {
           condition(expr: 'every 10 seconds')
           action(expr: 'ThingsGraph.startFlow(\"example01Flow\", bindings[name == \"device1\"].deviceId, bindings[name == \"device2\"].deviceId)')
         }
       }
     }
   }
   ```

1. **Create the flow configuration\.**

   After you define the flow configuration, you create it by using the `CreateSystemInstance` API\. This API consumes a JSON object that contains two parameter values: `language` and `text`\. 

   Currently the only supported value for `language` is GraphQL\. The value of `text` is a set of TDM definitions implemented in GraphQL\. 

   The resulting JSON looks like the following example\.

   ```
   {
       "language": "GRAPHQL",
       "text": "{ query Example01 @deployment(id: 'urn:tdm:REGION/ACCOUNT ID/default:deployment:Example01', systemId: 'urn:tdm:REGION/ACCOUNT ID/default:system:Example01') {device1(deviceId: 'DeviceA') device2(deviceId: 'DeviceB') triggers {TenSecondTrigger(description: 'a trigger') {condition(expr: 'every 10 seconds') action(expr: 'ThingsGraph.startFlow(\"example01Flow\", bindings[name == \"device1\"].deviceId, bindings[name == \"device2\"].deviceId)')}}}}"
   }
   ```

   For cloud deployments, the `CreateSystemInstance` API also requires two additional parameters:  
`target`  
The target type for your deployment\. For cloud deployments, specify `CLOUD`\.  
`flowActionsRoleArn`  
 The ARN of the IAM role that AWS IoT Things Graph assumes when it executes the flow\. This role must have read and write access to Lambda and AWS IoT and any other AWS services that the flow uses when it executes\. 

   After you construct the JSON payload that contains the GraphQL definition of the flow configuration, upload it to your namespace by using the following CLI command\. The output of this command includes an ID for your flow configuration\. You use this value when you deploy the flow configuration\.

   ```
   aws iotthingsgraph create-system-instance --definition file://flow configuration file name --target CLOUD --flow-actions-role-arn "IAM Role"  
   ```

1. **Deploy the flow configuration\.**

   Deploy your flow configuration to the cloud by using the `DeploySystemInstance` API\.

   The following command shows how to deploy a flow configuration by using the AWS CLI\.

   ```
   aws iotthingsgraph deploy-system-instance --id SYSTEM DEPLOYMENT CONFIGURATION ID
   ```

## Create and Deploy a Flow \(AWS IoT Things Graph console\)<a name="iot-tg-workflows-cloud-console"></a>

1. Sign in to the [AWS IoT Things Graph console](https://console.aws.amazon.com/thingsgraph/home)\.

   Choose **Create flow**\.  
![\[Choose the Create flow button at the upper right of the page.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGConsoleCreateFlow.png)

1. **Create a flow\.**

   In the **Flow configuration** pane, enter a name for your flow\. This name can't contain any spaces\.

   Choose **Create flow**\.  
![\[Flow name is ReadBarCode.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGCreateFlowExample.png)

1. **Add the trigger and devices to the flow\.**

   On the **Logic** tab, choose **Clock**, and then drag it into the flow designer\. 

   Search for **DeviceA**, which is a barcode reader that you created in [Creating and Uploading Models](iot-tg-models-gs.html)\. Select the device and drag it into the flow designer\. Do the same for **DeviceB**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGFlowConfigExample.png)

1. **Connect the devices\.**

   In the flow designer, select the edge of **ClockTrigger** and connect it to **DeviceA**\. Connect **DeviceA** and **DeviceB** in the same way\.  
![\[ClockTrigger, DeviceA, and DeviceB connected by straight lines.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGFlowConfigExample2.png)

1. **Update the **ClockTrigger**\.**

   In the trigger editor that appears in the right pane, for **Frequency**, enter 10, and then select seconds from the menu on the right\. For **Action**, choose **ThingsGraph\.startFlow**\.   
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGFlowConfigExample3.png)

1. **Update the device action for **DeviceA**\.**

   In the flow designer, select **DeviceB**\. Select **No action configured** in the action editor that appears in the right pane\. Select **readBarcode** from the drop\-down box that appears\. Select the arrow that appears to the left of the **Output** option\. Enter **deviceAResult** in the text box that appears under **Output**\.

   Click the surface of the flow designer to close the action editor\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDeviceActivityExample.png)

1. **Update the device action for **DeviceB**\.**

   In the flow designer, select **DeviceB**\. Select **No action configured** in the action editor that appears in the right pane\. Select **doSomething** from the drop\-down box that appears\. Select **Define Input** under the **Inputs** option\. Enter **$\{deviceAResult\}** in the pop\-up box that appears\. Select the **Save Input** button

   Click the surface of the flow designer to close the action editor\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDeviceActivityExample2.png)

1. **Publish the flow\.**

   Choose **Publish** at the upper right of the page\. This creates the flow and adds it to the list of flows that can be deployed\. Then choose **Go to flow list** to verify that your flow is created\.  
![\[Buttons for publishing the flow and then navigating to the flow list page.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGPublishFlowExample.png)

1. **Start creating the flow configuration\.**

    On the **Flows** list page, select the check box next to the flow that you just created, and then choose **Create flow configuration**\.  
![\[When the check box next to the flow is checked, the Create flow configuration button appears.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGCreateDeploymentExample.png)

1. **Name the flow configuration\.**

   On the **Describe flow configuration** page, select your flow and enter a deployment name\. The deployment name can't contain spaces\. Choose **Cloud**, and then choose **Next**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGExampleCreateCloudDeployment.png)

1. **Configure the target\.**

   On the **Configure target** page, enter the ARN of the role that you created in [Prepare for Cloud Deployments](iot-tg-gs-environment-cloud.html)\. Choose **Next**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGExampleCreateCloudDeployment2.png)

1. **Select things\.**

   On the **Map Things** page, from the menu under **deviceA**, select the thing that you associated with this device in [Creating and Uploading Models](iot-tg-models-gs.html)\. Do the same for **deviceB**\. Choose **Next**\.  
![\[Associated things appear in drop-down menus under devices. Select the associated thing for each device.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGExampleConfigureDeployment.png)

1. **View the trigger\.**

   On the **Set up triggers** page, the following GraphQL appears in the editor\. This GraphQL specifies the time intervals at which the flow runs\. This flow runs every 10 seconds\. You don't need to edit this code\.

   Choose **Review**\.  
![\[Trigger definition that starts the flow every 10 seconds.\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDefineTriggerExample.png)

1. **Review and create**

   On the **Review and create** page, review the information you entered for your flow configuration\.

   Choose **Create**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGBarCodeReviewCreate.png)

1. Deploy

   When the **Flow configuration created** message appears, choose **Deploy now**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDeploymentCreated.png)

   After a successful deployment, the **Deployments** page displays **Deployed in target** in the **Status** column\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/thingsgraph/latest/ug/images/TGDeploySuccess.png)