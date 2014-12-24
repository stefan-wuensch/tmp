:one:

This document is a tutorial on how to set up the website stack used in the 
[CloudWatch Alarms with Nagios boot-camp](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/README.md).

**This procedure should ONLY be used if the existing demo website stack used in the bootcamp tutorial needs to be re-created.**



**Note**: If you need to re-create the CloudFormation Stack for the SNS Topics, the process is **exactly the same** 
except you will use the template 
[HUIT_Nagios_SNS_Topics_template.json](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/JSON-Templates/HUIT_Nagios_SNS_Topics_template.json) 
and the name of the stack _does not matter_. It can be named anything that makes sense (like `SNS-Topics-for-Nagios-CloudWatch-Bootcamp` or similar).





## Download the template

Download a copy of the website stack template 
[Boot-camp website stack 2014-12-23.json](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/JSON-Templates/Boot-camp%20website%20stack%202014-12-23.json)

You should not need to edit it. All the default values are populated to make it ready-to-go for the bootcamp tutorial.



## Create the stack

Go to CloudFormation Stacks console: <br>
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active

Click **Create Stack**:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-1.png)

**Enter a name for your Stack**. The name of the site stack for the bootcamp tutorial should be `HUIT-Nagios-CloudWatch-Bootcamp`

Next **Upload a template to Amazon S3**:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-2.png)

Click the **Choose File** button and select your new Template (from where you saved above) and **click Next**.

**Verify the Parameter values** to be what you entered into Defaults in your Template.

You should not need to change any values!

**Click Next**.

**Leave the Options as they are.** Just click **Next**.

If everything looks correct, **click Create**.

After a few minutes your new Stack should go from `CREATE_IN_PROGRESS` to `CREATE_COMPLETE`. If needed, click the Refresh button.

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-4.png)


