--------

--------

# Security for AWS IoT Greengrass Deployments<a name="iot-tg-security"></a>

AWS IoT Things Graph is deployed to AWS IoT Greengrass, and it stores flow configurations in Amazon Simple Storage Service\. This topic describes the security implications of this architecture and the security features directly related to AWS IoT Things Graph\.

## Control Plane<a name="iot-tg-security-control"></a>

The AWS IoT Things Graph control plane uses IAM policies to define what a user can do\. You can create policies that assign permissions for every available action in the [AWS IoT Things Graph API](https://docs.aws.amazon.com/thingsgraph/latest/APIReference/API_Operations.html)\. 

For example, the following policy assigns permission for a user to use the [CreateSystemInstance](API_CreateSystemInstance.html) API\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iotthingsgraph:CreateSystemInstance"
            ],
            "Resource": [
                "*"
            ]
        }
}
```

The following policy gives permission to perform all actions\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iotthingsgraph:*"
            ],
            "Resource": [
                "*"
            ]
        }
       ]
}
```

Additionally, users should have permissions to perform the following actions:
+ `s3:REST.GET.OBJECT`
+ `s3:REST.PUT.OBJECT`
+ `iot:CreateTopicRule`
+ `iot:ListTopicRules`
+ `iot:GetTopicRule`
+ `iot:DescribeThing`
+ `iot:ListThingPrincipals`
+ `iot:UpdateThing`
+ `iot:AddThingToThingGroup`
+ `iot:DescribeThingGroup`
+ `iot:CreateThingGroup`
+ `iot:ListThingsInThingGroup`
+ `iot:DeleteThingGroup`
+ `iot:RemoveThingFromThingGroup`
+ `greengrass:CreateGroupVersion`
+ `greengrass:CreateDeployment`
+ `greengrass:GetDeviceDefinitionVersion`
+ `greengrass:CreateDeviceDefinitionVersion`
+ `greengrass:CreateDeviceDefinition`
+ `greengrass:GetFunctionDefinitionVersion`
+ `greengrass:CreateFunctionDefinitionVersion`
+ `greengrass:CreateFunctionDefinition`
+ `greengrass:GetResourceDefinitionVersion`
+ `greengrass:CreateResourceDefinitionVersion`
+ `greengrass:CreateResourceDefinition`
+ `greengrass:GetSubscriptionDefinitionVersion`
+ `greengrass:CreateSubscriptionDefinitionVersion`
+ `greengrass:CreateSubscriptionDefinition`
+ `greengrass:ListGroups`
+ `greengrass:GetGroupVersion`
+ `greengrass:GetConnectorDefinitionVersion`
+ `greengrass:CreateConnectorDefinition`
+ `greengrass:CreateConnectorDefinitionVersion`
+ `iam:PassRole`

**Namespaces**

Every entity that you use in a workflow must belong to your namespace\. See [Namespaces](iot-tg-whatis-namespace.html) for more information\.

Entities stored in an account's namespace are available to all users in an account\. All users in the account have read and write access to all entity definitions in the namespace\.

Because you deploy AWS IoT Things Graph to AWS IoT Greengrass, you need to set up AWS IoT Greengrass security by following the steps in [Configuring Greengrass Security](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html#gg-config-sec) in the *AWS IoT Greengrass Developer Guide*\. When you deploy a workflow, you provide AWS IoT Things Graph with an IAM service role\. This role should contain all of the policies required for AWS IoT Things Graph to publish and subscribe to all of the MQTT topics that are used in the workflow\. For more information about IAM roles, see [Using IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html)\.

## Data Plane<a name="iot-tg-security-data"></a>

AWS IoT Things Graph runs as an AWS Lambda function on the AWS IoT Greengrass core device\. When new workflows are deployed to AWS IoT Greengrass, AWS IoT Things Graph creates entries in the subscription table for communication between the devices in the workflow and the AWS IoT Things Graph Lambda function\. 

AWS IoT Things Graph therefore creates subscriptions to all of the MQTT topics that the devices in your workflow use\. The subscription table is used by the AWS IoT Greengrass core device for authorization of all communications\. The AWS IoT Things Graph Lambda function mediates communications among the devices in the workflow\.