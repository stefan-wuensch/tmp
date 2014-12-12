This file **README.md** is a step-by-step guide to setting up AWS CloudWatch Alarms that can be integrated with Nagios.

This is a high-level tutorial, designed to be only what you need to get started. For detailed information, see
https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Detailed_Tutorial.md



# Gathering Parameters

The following Parameter values need to be collected from the web site Stack that is to be monitored.


### 1. **AdminNode**

This is the EC2 Instance ID of the Web Admin Node for the site. 

Start at https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:sort=tag:Name

Depending on how the site owner has set up the stack, the name may be easy to find or not easy at all. In this example 
the Instance name makes it easy: `HPACNewsProd Web Admin Server`

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/admin_node.png)

The `Instance ID` is in the format `i-abcd1234` (letter 'i', dash, and eight random alphanumeric characters).

In this example the Instance ID is `i-97da517d`. **Save that for later!**




### 2. **AutoScalingGroupMinSize**

This is the minimum number of EC2 Instances the site will have. 

Go to EC2 Auto Scaling Groups: <br>
https://console.aws.amazon.com/ec2/autoscaling/home?region=us-east-1#AutoScalingGroups:view=details

Find the name of the site:
![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/auto-scaling-min.png)

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/auto-scaling-min-2.png)

**Jot down the Min value for that group.** In this case, it's 6.




### 3. **CriticalAlarmTopic**

This is the SNS Topic Name for getting the Alarm state to Nagios. 

Go to https://console.aws.amazon.com/sns/home?region=us-east-1# and expand the list of Topics:
![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/sns-topic-1.png)

Select the Topic that refers to **HUIT Nagios Critical**.

You can confirm that this is the correct SNS Topic by making sure the Endpoint URL is production Nagios, either:<br>
https://nagios.fas.harvard.edu/aws_sns_receiver.php<br>
or<br>
https://nagios.huit.harvard.edu/aws_sns_receiver.php

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/sns-topic-2.png)

**Copy the Topic ARN.**
In this example, it's `arn:aws:sns:us-east-1:014311208322:HUIT_Nagios_Critical`





### 4. **MasterDB**

This is the RDS Instance ID.

Go to the RDS Dashboard list of Instances: <br>
https://console.aws.amazon.com/rds/home?region=us-east-1#dbinstances:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-1.png)

It can be tricky to figure out which RDS Instance is the one we want. If you can't tell which one it is 
we'll look at the site Stack for help. 

**In a new browser tab or window**, go to CloudFormation Stacks: <br>
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active

Select the site stack, go to **Resources** and you will find `DBInstance` in the _Logical ID_ column. 

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-2.png)

**Copy the _Physical ID_ for that DBInstance** (in this example `hdjhee21ma2vq6`).




### 5. **DBClass**

This is the Database Instance Class. 

Now that we know the RDS Instance ID (from Step 4), go back to the **other browser tab / window** with the RDS Instances list and find that ID on the page:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-3.png)

**Copy the value shown in the Class column for that DB Instance.**

In this example it's `db.m3.large`





### 6. **DBName**

This is the application-layer database name. 

Since we have already identified which RDS Instance is the correct one (in step #4 above), we now know where to find the name of the database.

Select (click) that RDS Instance. The full detail for that RDS Instance will be shown.

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-4.png)

**Copy the DB Name.**

In this example it's `wordpressdb`





### 7. **ElasticLoadBalancer**

This is the name of the ELB (Elastic Load Balancer).

Go to the EC2 Dashboard section on **Load Balancers**: <br>
https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LoadBalancers:

Finding the correct ELB can be difficult, just like finding the correct RDS in Step 4. 

Just as with Step 4: if you can't easily figure out which ELB is the right one, refer to the site Stack.

**In a new browser tab or window**, go to CloudFormation Stacks: <br>
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active <br>
(Of course if you kept the browser tab or window open from Step 4 you can just switch back to it.)

Select the site stack, go to **Resources** and you will find `ElasticLoadBalancer` in the _Logical ID_ column. 

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/elb-1.png)

**Copy the _Physical ID_ for that ElasticLoadBalancer** (in this example `HPACNewsP-ElasticL-KUWSX4IQHDSD`).






# Starting the Alarms Stack


# Configuring Nagios


# Testing


