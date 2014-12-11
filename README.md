This file README.md is a step-by-step guide to setting up AWS CloudWatch Alarms that can be integrated with Nagios.

This is a high-level tutorial, designed to be only what you need to get started. For detailed information, see
https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Detailed_Tutorial.md



# Gathering Parameters

The following Parameter values need to be collected from the web site Stack that is to be monitored.


## 1. **AdminNode**

This is the EC2 Instance ID of the Web Admin Node for the site. 

Start at https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:sort=tag:Name

Depending on how the site owner has set up the stack, the name may be easy to find or not easy at all. In this example 
the Instance name makes it easy: `HPACNewsProd Web Admin Server`

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/admin_node.png)

The `Instance ID` is in the format `i-abcd1234` (letter 'i', dash, and eight random alphanumeric characters).

In this example the Instance ID is `i-97da517d`. Write that down!



2. **AutoScalingGroupMinSize**

This is the minimum number of EC2 Instances the site will have. 

Go to EC2 Auto Scaling Groups: 
https://console.aws.amazon.com/ec2/autoscaling/home?region=us-east-1#AutoScalingGroups:view=details

Find the name of the site:
![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/auto-scaling-min.png)

Write down the Min value for that group:
![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/auto-scaling-min-2.png)







# Starting the Alarms Stack


# Configuring Nagios


# Testing


