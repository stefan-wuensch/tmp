:one:

This document is a tutorial on how to set up the website stack used in the 
[CloudWatch Alarms with Nagios boot-camp](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/README.md).

**This procedure should ONLY be used if the existing demo website stack used in the bootcamp tutorial needs to be re-created.**

## Download the template

Download a copy of [Boot-camp website stack 2014-12-23.json](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/JSON-Templates/Boot-camp%20website%20stack%202014-12-23.json)

You should not need to edit it. All the default values are populated to make it ready-to-go for the bootcamp tutorial.



## Create the stack

Go to CloudFormation Stacks console: <br>
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active

Click **Create Stack**:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-1.png)

**Enter a name for your Stack**, and then **Upload a template to Amazon S3**:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-2.png)

Click the **Choose File** button and select your new Template (from where you saved it in Step 1) and **click Next**.

