# jenkins-ec2-fleet-slaves

### we need to create on-demand slaves for jenkins master which can be scaled and managed using aws ec2-fleet plugin

- if we create slaves manually, and if that instance got lost, then we cant replace it another one automatically. so we need to handle this situation
- we can use aws ec2-fleet, fleet means combination of spot,on-demand,reserved instances, fleet will replace another instance if slave is terminated or removed
- while launching a single on-demand instance manually may be straightforward, EC2 Fleet provides a more robust and scalable solution for managing fleets of instances, especially in large-scale or dynamic environments where automation, flexibility, and efficiency are critical.
- Launching a single on-demand instance manually versus using EC2 Fleet may not show significant differences, but the strength of EC2 Fleet lies in its ability to efficiently manage and provision multiple instances at scale, particularly across various instance types, pricing models, and availability zones.
- you can achieve a mix of on-demand and spot instances within a single EC2 Fleet. EC2 Fleet is a feature that allows you to provision a combination of on-demand, spot, and reserved instances to meet a specified target capacity.
- you cannot control ec2-fleet with aws management console, you dont have an option to it, need to manage from awscli only, you can only have console access for spot fleet.
- for ec2-fleet you dont have aws GUI, you need to create and control via awscli.
- create a launch template and dont specify instance types there, in the fleet we will specify the instnace types.
- if an instnace got launched, then if we remove that then again fleet will launch another ec2 instnace.
- it will take some time and lauch ondemand ec2 instnace, according to lowest price it will launch...we have selected by default lowest price.
- now we can add this fleet for jenkins master server, so master server can add or scale slaves nodes from this fleet...so jenkins master needs to have permissions to do that...so it needs ec2 full access or fleet access role for jenkins mater
- WE CAN ALSO INCLUDE SPOT INSTNACES + ON DEMAND + RESERVED INSTNACES INSIDE FLEET, if spot instnace is launched, then in spot requests, one spot request will be created.


```
on-demand instances within a spot request provides a balance between cost optimization and reliability, allowing you to take advantage of spot pricing while maintaining the assurance of on-demand capacity for critical workloads.

When you launch an on-demand EC2 instance via a spot request with a maximum price equivalent to the on-demand price, AWS treats it like a spot instance with a maximum bid equal to the on-demand price. This means that as long as the spot price remains below the on-demand price, you'll continue to benefit from the lower spot price. However, if the spot price exceeds the on-demand price, your instance could be terminated, similar to how traditional spot instances behave.

So while launching an on-demand instance via spot request can provide cost savings when spot prices are low, there's a risk of termination if spot prices rise above the on-demand price. This is an important consideration when deciding whether to use this approach, particularly for workloads that require consistent availability and cannot tolerate interruptions.


Suppose you want to launch an on-demand t2.micro instance on AWS. The regular on-demand price for a t2.micro instance is $0.018 per hour.

Instead of launching it directly as an on-demand instance, you decide to use a spot request with a maximum price set to the on-demand price, which is $0.018 per hour.

Here's how it works:

Spot Price Dynamics: Spot prices for instances fluctuate based on supply and demand in AWS data centers. At certain times, spot prices might be lower than the on-demand price due to available spare capacity.
Scenario 1: Spot Price Below On-Demand Price: If the spot price for a t2.micro instance remains below $0.018 per hour, your spot request will be fulfilled with a spot instance at the lower spot price. You'll benefit from cost savings compared to running the instance at the regular on-demand price.
Scenario 2: Spot Price Exceeds On-Demand Price: However, if the spot price for a t2.micro instance rises above $0.018 per hour, your spot instance could be terminated. This is because your spot request has a maximum price equivalent to the on-demand price, and AWS will reclaim spot instances if their price exceeds your maximum bid. In this scenario, you might experience interruption to your instance if it gets terminated due to spot price fluctuations.
So, while using on-demand instances within a spot request can provide cost savings when spot prices are low, there's a risk of termination if spot prices rise above the on-demand price. This risk should be considered, especially for workloads that require consistent availability and cannot tolerate interruptions.

Let's say you have a workload where you need to launch 5 t2.micro instances:

On-Demand Capacity: Since the allocation strategy ensures that on-demand capacity is always fulfilled, AWS will immediately provision the required number of instances as on-demand instances. In this case, let's say you requested 2 instances as on-demand.
Spot Capacity: The remaining capacity beyond the on-demand portion of your request will be fulfilled with spot instances if there is capacity and availability. Continuing with the example, if you requested a total of 5 instances and 2 were fulfilled as on-demand, the remaining 3 instances will be launched as spot instances if there is available capacity and the spot price is within your specified maximum price.

So, in this scenario:

You have the assurance of immediate on-demand capacity for a portion of your instances, ensuring availability and reliability for critical workloads.
The remaining capacity is fulfilled with potentially cheaper spot instances if available, optimizing costs for non-critical or flexible workloads.

This allocation strategy provides a balance between reliability and cost optimization by leveraging both on-demand and spot instances based on your workload requirements and cost considerations.

When you launch an on-demand instance through a spot request, it might seem counterintuitive, but the instance's lifecycle will still be categorized as "on-demand" rather than "spot." This distinction is because you've initiated the instance as an on-demand instance, even though you've used the spot request mechanism to launch it.

In the AWS Management Console or through the AWS CLI/API, when you inspect the details of the instance, you'll typically see it labeled as an on-demand instance. This means that the instance is being billed at the on-demand rate, and it follows the same lifecycle characteristics as a regular on-demand instance.

However, it's essential to remember that even though you've launched an on-demand instance through a spot request, the pricing model still behaves like a spot instance with a maximum bid set to the on-demand price. So, while the instance is classified as on-demand, it might be subject to termination if the spot price exceeds the on-demand price.

In summary, the instance's lifecycle will be labeled as on-demand, but its pricing and potential for termination will follow the behavior of a spot instance based on the spot price fluctuations.


there isn't a direct way to specify a mix of on-demand and spot instances within the same spot request. Spot requests are primarily intended for spot instances, and there isn't a built-in mechanism to mix on-demand and spot instances within a single spot request.

To achieve the scenario of launching one on-demand and one spot instance simultaneously, you would typically need to make separate requests—one for the on-demand instance and one for the spot instance.

So, you'd need to run two separate commands: one for launching the on-demand instance and another for creating a spot request to launch the spot instance. There isn't a single command that can handle both types of instances within the same spot request.

Using boto3, the approach is similar to the AWS CLI. There isn't a direct way to specify a mix of on-demand and spot instances within the same spot request. You would still need to make separate requests to launch on-demand and spot instances.

For example, if you need to launch a total of 5 instances (2 on-demand and 3 spot), you would make two separate requests:

- One request to launch 2 on-demand instances using the run_instances method.
- Another request to create a spot request for 3 spot instances using the request_spot_instances method.

These requests would be made independently, and the instances launched would not necessarily be related to the same spot request. Each instance launched, whether on-demand or spot, would have its own lifecycle and attributes.

EC2-FLEET:
==========
you can achieve a mix of on-demand and spot instances within a single EC2 Fleet. EC2 Fleet is a feature that allows you to provision a combination of on-demand, spot, and reserved instances to meet a specified target capacity.

Using EC2 Fleet, you can define a launch template or launch configuration that specifies the instance types, pricing models (on-demand, spot), and other parameters for your fleet. You can then specify a target capacity, and EC2 Fleet will provision a mix of instance types and pricing models to meet that capacity based on your configuration.

aws ec2 create-fleet \
    --launch-template-configurations LaunchTemplateName=<your-launch-template>,WeightedCapacity=2 \
    --target-capacity-specification TotalTargetCapacity=5,OnDemandTargetCapacity=2,SpotTargetCapacity=3 \
    --region us-west-2

Launching a single on-demand instance manually versus using EC2 Fleet may not show significant differences, but the strength of EC2 Fleet lies in its ability to efficiently manage and provision multiple instances at scale, particularly across various instance types, pricing models, and availability zones.

Here are some scenarios where EC2 Fleet shines:

Managing Large Workloads: If you need to provision a large number of instances, possibly across multiple instance types or availability zones, EC2 Fleet simplifies the process by allowing you to define a single configuration for your fleet and automatically managing the distribution and provisioning of instances.
Mixed Instance Types and Pricing Models: EC2 Fleet allows you to mix and match different instance types (e.g., t2.micro, m5.large) and pricing models (on-demand, spot) within the same fleet. This flexibility enables you to optimize costs and performance based on your workload requirements.
Automatic Scaling: EC2 Fleet integrates seamlessly with Auto Scaling, enabling you to automatically scale your fleet based on demand. You can define scaling policies to adjust the fleet's capacity in response to changing workload conditions, ensuring that you have the right number of instances available at all times.
Spot Instance Management: If you're using spot instances, EC2 Fleet offers advanced features for managing spot capacity efficiently, such as spot capacity rebalancing, capacity-optimized allocation strategies, and fleet diversification. These features help maximize spot instance availability and reduce interruptions due to spot price fluctuations.
Simplified Management: With EC2 Fleet, you can manage your fleet using a single configuration, making it easier to provision, monitor, and manage your instances. EC2 Fleet handles the complexities of managing multiple instance launches and instance types, freeing you from manual intervention.
In summary, while launching a single on-demand instance manually may be straightforward, EC2 Fleet provides a more robust and scalable solution for managing fleets of instances, especially in large-scale or dynamic environments where automation, flexibility, and efficiency are critical.

- you cannot control ec2-fleet with aws management console, you dont have an option to it.....you can only have console access for spot fleet
- for ec2-fleet you have aws GUI, you need to create and control via cli-input-json
- create a launch template and dont specify instance types there, in the fleet we will specify the instnace types
- if an instnace got launched, then if we remove that then again fleet will launch another ec2 instnace
- it will take some time and lauch ondemand ec2 instnace, according to lowest price it will launch...we have selected by default lowest price
- now we can add this fleet for jenkins master server, so master server can add or scale slaves nodes from this fleet...so jenkins master needs to have permissions to do that...so it needs ec2 full access or fleet access role
- WE CAN ALSO INCLUDE SPOT INSTNACES + ON DEMAND + RESERVED INSTNACES INSIDE FLEET, if spot instnace is launched, then in spot requests, one spot request will be created.

Yes, the fleet will launch instances based on the combination of instance types and their respective weighted capacities specified in the overrides. However, the exact instance launched will depend on various factors including current spot prices, instance availability, and your fleet's configuration settings such as the allocation strategy.

In your configuration, you have specified multiple instance types with the same weighted capacity of 1, indicating that each instance type has an equal chance of being launched. However, since you have mentioned that you want to launch the "low-cost instance," it's important to consider the spot prices and instance types you've included in the fleet.

If the spot price for c4.large instances is lower compared to m5.large instances at the time of fleet creation, there's a higher chance that c4.large instances will be launched. However, if the spot price for m5.large instances is lower, then m5.large instances may be launched instead.

To prioritize launching the lowest-cost instances, you might want to adjust the fleet configuration to include only the instance types with the lowest spot prices at the time of fleet creation. Additionally, you could consider using the "lowestPrice" allocation strategy for spot fleets to optimize cost further.


In your case, you have configured the fleet to use on-demand instances, which means the instances will continue to run as long as the fleet is active and there is sufficient capacity available in your AWS account. On-demand instances do not have a predefined termination time and will continue to run until they are explicitly stopped or terminated.


```

```
we can scale jenkins slaves using two plugins
1.EC2-Fleet-Plugin	
2.EC2-Plugin
```

- create a launch template with default vpc and subnet:
- take amazonlinux2 ami, make sure java 17 is installed
- create a role with ec2 full access

```
- create a launch template for fleet
cat > launch-template-config.json

{
    "ImageId": "ami-0395649fbe870727e",
    "KeyName": "JenkinsSlave",
    "NetworkInterfaces": [
        {
            "DeviceIndex": 0,
            "SubnetId": "subnet-0ac24905670f08c76",
            "Groups": ["sg-0c168caf30a86e65b"],
            "AssociatePublicIpAddress": true,
            "DeleteOnTermination": true
        }
    ],
    "IamInstanceProfile": {
        "Arn": "arn:aws:iam::732530106046:instance-profile/vijay-role"
    },
    "TagSpecifications": [
        {
            "ResourceType": "instance",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "VijayInstanceFromFleet"
                }
            ]

```

```
aws ec2 create-launch-template \
    --launch-template-name VijayLaunchTemplateForFleet \
    --version-description "1" \
    --tag-specifications 'ResourceType=launch-template,Tags=[{Key=Name,Value=VijayLaunchTemplateForFleet}]' \
    --launch-template-data file://launch-template-config.json \
    --region us-west-2 --profile vijay


{
    "LaunchTemplate": {
        "LaunchTemplateId": "lt-02e17929b5868f792",
        "LaunchTemplateName": "sample-launch-template",
        "CreateTime": "2024-05-08T11:28:36+00:00",
        "CreatedBy": "arn:aws:iam::732530106046:user/vijay",
        "DefaultVersionNumber": 1,
        "LatestVersionNumber": 1,
        "Tags": [
            {
                "Key": "Name",
                "Value": "SampleLaunchTemplate"
            }
        ]
    }
}
```

```
- create a fleet using launch template id

cat > config.json
{
    "LaunchTemplateConfigs": [
        {
            "LaunchTemplateSpecification": {
                "LaunchTemplateId": "lt-02e17929b5868f792",
                "Version": "1"
            },
            "Overrides": [
                {
                    "InstanceType": "c4.xlarge",
                    "WeightedCapacity": 1
                },
                {
                    "InstanceType": "m5.large",
                    "WeightedCapacity": 1
                },
                {
                    "InstanceType": "c4.large",
                    "WeightedCapacity": 1
                }
            ]
        }
    ],
    "TargetCapacitySpecification": {
        "TotalTargetCapacity": 1,
        "OnDemandTargetCapacity": 1,
        "DefaultTargetCapacityType": "on-demand"
    }
}

on-demand
spot

aws ec2 create-fleet --cli-input-json file://config.json --profile vijay --region us-west-2

{
    "FleetId": "fleet-abbfb319-5c69-43e6-aa95-9c46a8e04b53"
}

-- goto ec2 instances, after sometime, you can see ec2 instance running in us-west-2

SampleLaunchTemplate  i-087d17d467f97598d	 Running  m5.large	  2/2 checks passed	  View alarms

aws ec2 describe-fleets --fleet-id fleet-abbfb319-5c69-43e6-aa95-9c46a8e04b53 --profile vijay --region us-west-2

aws ec2 describe-fleets --profile vijay    --- lists all fleets

aws ec2 delete-fleets --fleet-id fleet-abbfb319-5c69-43e6-aa95-9c46a8e04b53 --terminate-instances --profile vijay --region us-west-2

```

```
JENKINS TEST SERVER LOGS WILL BE PRESESNT IN :
/var/log/jenkins
tail -f jenkins.log
```

- now we need to configure this instance as a slave machine for jenkins master
- we need EC2-Fleet plugin to be installed on jenkins

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/285862a7-604e-4716-a21c-37c09d7b72fb)


- configure cloud

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/b7473fad-561b-45af-9092-7035c0c3d6ff)

- here i have given ec2 full access to jenkins master, so no need aws credentials

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/9b4013e6-8a8f-4d4d-91a1-3e1afec9f775)

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/acc24253-87a4-41fb-8be0-91c697d8a64f)

- if you are under vpn, you can select private ip communication

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/c544383f-9d80-46eb-81a9-4f58e160f9fa)

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/12194ac8-8d7e-472b-8765-1d3925a29d05)

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/1b06285b-27f9-40bd-bffb-7e7051c973be)

- rest options keep as default
- save

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/46a6d0a6-9da1-445f-a867-ccb322afaf05)


- create a new free stylejob for testing

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/7b2e8ac3-60dc-4f08-9d68-7091d7993e97)

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/ddbab63e-c172-4f8e-b33a-971234e9a2d4)

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/76a1498e-4177-4e5d-a8b9-6c9cf6c9c409)

- apply and save

![image](https://github.com/vijay2181/jenkins-ec2-fleet-slaves/assets/66196388/b73463a8-7854-463a-9e12-e42e429d3868)



























