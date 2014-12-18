This file **README.md** is a step-by-step guide to setting up AWS CloudWatch Alarms that can be integrated with Nagios.

This is a high-level tutorial, designed to be only what you need to get started. For detailed information, see
https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Detailed_Tutorial.md

**Note**: All the images presented here can be viewed in their native size by clicking the image and then 
clicking **Raw**.



# Gathering Parameters

The following Parameter values need to be collected from the web site Stack that is to be monitored.

All of the following **eight** Parameters will be used as inputs to the Alarm Stack creation.

1. AdminNode
2. AutoScalingGroupMinSize
3. CriticalAlarmTopic
4. MasterDB
5. DBClass
6. DBName
7. ElasticLoadBalancer
8. SiteName


### 1. **AdminNode**

This is the EC2 Instance ID of the Web Admin Node for the site. 

Start at https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:sort=tag:Name

Depending on how the site owner has set up the stack, the name may be easy to find or not easy at all. In this example 
the Instance name makes it easy: `HPACNewsProd Web Admin Server`

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/admin_node.png)

If the Web Admin Instance cannot be easily identified, _you may need to contact the customer_ and have them tell you its name.

The `Instance ID` is in the format `i-abcd1234` (letter 'i', dash, and eight random alphanumeric characters).

In this example the Instance ID is `i-97da517d`. **Save that for later!**




### 2. **AutoScalingGroupMinSize**

This is the minimum number of EC2 **webserver** Instances the site will have. <br>(This is **not** to do with the Admin Node.)

Go to EC2 Auto Scaling Groups: <br>
https://console.aws.amazon.com/ec2/autoscaling/home?region=us-east-1#AutoScalingGroups:view=details

Find the name of the site:
![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/auto-scaling-min.png)

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/auto-scaling-min-2.png)

**Jot down the Min value for that group.** In this case, it's `6`.




### 3. **CriticalAlarmTopic**

This is the SNS Topic Name for getting the Alarm state to Nagios. 

Go to https://console.aws.amazon.com/sns/home?region=us-east-1# and expand the list of Topics:
![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/sns-topic-1.png)

Select the Topic that refers to **HUIT Nagios Critical**.

You can confirm that this is the correct SNS Topic by making sure the Endpoint URL is production Nagios, either:<br>
`https://nagios.fas.harvard.edu/aws_sns_receiver.php`<br>
or<br>
`https://nagios.huit.harvard.edu/aws_sns_receiver.php`

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/sns-topic-2.png)

**Copy the Topic ARN.**
In this example, it's `arn:aws:sns:us-east-1:014311208322:HUIT_Nagios_Critical`





### 4. **MasterDB**

This is the RDS Instance ID.

Go to the RDS Dashboard list of Instances: <br>
https://console.aws.amazon.com/rds/home?region=us-east-1#dbinstances:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-1.png)

It can be tricky to figure out which RDS Instance is the one we want. (The image above demonstrates this. There is no way 
to tell from what is listed here which of these RDS Instances is the right one!)

If you can't tell which one it is - like in this example - we'll look at the website CloudFormation Stack for help. 

**In a new browser tab or window**, go to CloudFormation Stacks: <br>
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active

Select the site stack, go to **Resources** and look for `AWS::RDS::DBInstance` in the _Type_ column.

_Some sites may only have one RDS Instance_, like this:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-2a.png)

In that case, **copy the _Physical ID_ for that DBInstance** (in this example `hdjhee21ma2vq6`). That's your value for **MasterDB**.

However, _sites can have more than one RDS Instance_... like a Master and a Replica. In those cases the customer has to 
clearly indicate which one is which. If they do not, then just as with the AdminNode in Step 1 _you will have to 
have the customer tell you_ which RDS Instance is the Master.

(Monitoring a Replica RDS can be done as well, but it's not included in this document.)

This is an example of a website Stack with a clearly-marked Master and Replica RDS Instance:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-2b.png)

In this case **copy the _Physical ID_ for the MasterDB** (in this example `Stefan-DemoMasterDatabase`). That's your value for **MasterDB**.



### 5. **DBClass**

This is the Database Instance Class. 

Now that we know the RDS Instance ID (from Step 4), go back to the **other browser tab / window** with the RDS Instances list and find that ID on the page:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-3.png)

In Step #4 we found the value for MasterDB is `hdjhee21ma2vq6`, so we're searching for that ID here.

**Copy the value shown in the Class column for that DB Instance.** <br>
In this example it's `db.m3.large`





### 6. **DBName**

This is the application-layer database name. 

Since we have already identified which RDS Instance is the correct one (in step #4 above), we now know where to find the name of the database.

Again for this example our MasterDB is `hdjhee21ma2vq6`.

Select (click) that RDS Instance. The full detail for that RDS Instance will be shown.

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/rds-4.png)

**Copy the DB Name.** <br>
In this example it's `wordpressdb`





### 7. **ElasticLoadBalancer**

This is the name of the ELB (Elastic Load Balancer).

Go to the EC2 Dashboard section on **Load Balancers**: <br>
https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LoadBalancers:

Finding the correct ELB can be difficult, just like finding the correct RDS in Step 4. 

Just as with Step 4: if you can't easily figure out which ELB is the right one, refer to the site Stack.

**In a new browser tab or window**, go to CloudFormation Stacks again: <br>
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active <br>
(Of course if you kept the browser tab or window open from Step 4 you can just switch back to it.)

Select the site stack, go to **Resources** and you will find `ElasticLoadBalancer` in the _Logical ID_ column. 

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/elb-1.png)

**Copy the _Physical ID_ for that ElasticLoadBalancer** <br>
(in this example `HPACNewsP-ElasticL-KUWSX4IQHDSD`).




### 8. SiteName

This is the name of the customer website, changed to be a unique name which will be used by Nagios.

**SiteName must be unique across all AWS VPCs and sites that are monitored by HUIT Nagios.** 

The format must be: website name as FQDN, with dots (periods) replaced by dashes (hyphens). 

Examples:

Public Site Name		| SiteName value in CloudFormation Template
------------------------------- | ------------------------------
`harvard.edu` 			| `harvard-edu`
`news.harvard.edu` 		| `news-harvard-edu`
`campaign.harvard.edu` 		| `campaign-harvard-edu`
`online-learning.harvard.edu` 	| `online-learning-harvard-edu`

You can choose any value for `SiteName` as long as it follows these guidelines and **is unique**.






# Starting the Alarms Stack


### 1. Create your template

For this example we will be starting with an existing *Stack Template* from HPAC / HWP.

**Download a copy** of the file 
[online-learning-harvard-edu.json](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/JSON-Templates/online-learning-harvard-edu.json) 
onto your local computer. You need to be able to upload the finished Template file to AWS via your web browser.

**Rename your copy** to something appropriate for this tutorial, such as `my-test-template.json`

In your favorite editor, **open your copy of the template**.

Each one of the eight Parameters you collected earlier will be entered as a **Default** in your new template.

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/template-1.png)

For example: using the Parameter values from the examples above, the Parameters in your new template would look like this:
```
  "Parameters" : {

	"ElasticLoadBalancer" : {
		"Description" : "The Elastic Load Balancer for the stack",
		"Type" : "String",
		"Default" : "HPACNewsP-ElasticL-KUWSX4IQHDSD"
	},

	"AdminNode" : {
		"Description" : "The Admin Node for the stack",
		"Type" : "String",
		"Default" : "i-97da517d"
	},

	"AutoScalingGroupMinSize" : {
		"Description" : "The minimum number of instances of the Auto Scaling Group for the stack",
		"Type" : "Number",
		"Default" : 6,
		"MinValue": 1,
		"ConstraintDescription": "Must be a number, one or greater"
	},

	"MasterDB" : {
		"Description" : "The RDS Instance Name for the database stack",
		"Type" : "String",
		"Default" : "hdjhee21ma2vq6"
	},

	"CriticalAlarmTopic" : {
		"Description" : "The name of the Nagios Critical Alarm SNS Topic",
		"Type" : "String",
		"Default" : "arn:aws:sns:us-east-1:014311208322:HUIT_Nagios_Critical"
	},

	"DBClass" : {
        	"Description" : "Database instance class",
      		"Type" : "String",
		"Default" : "db.m3.large",
		"AllowedValues" : [ 
					"db.m1.large", 
					"db.m1.small", 
					"db.m1.xlarge", 
					"db.m2.2xlarge", 
					"db.m2.4xlarge",
					"db.m2.xlarge", 
					"db.m3.large",
					"db.m3.medium",
					"db.t1.micro"
		],
      		"ConstraintDescription" : "must select a valid database instance type."
    	},


    	"DBName": {
      		"Description" : "The Drupal database name",
      		"Type": "String",
		"Default" : "wordpressdb",
      		"MinLength": "1",
      		"MaxLength": "64",
      		"AllowedPattern" : "[a-zA-Z][a-zA-Z0-9]*",
      		"ConstraintDescription" : "must begin with a letter and contain only alphanumeric characters."
    	},

    	"SiteName": {
      		"Description" : "Customer Site Name",
      		"Type": "String",
		"Default" : "aws-test-huit-harvard-edu",
      		"MinLength": "1",
      		"MaxLength": "64",
      		"AllowedPattern" : "[a-zA-Z][a-zA-Z0-9-]*",
      		"ConstraintDescription" : "must begin with a letter and contain only alphanumeric characters."
    	}
    },
```

Note that you **only need to change the values for the `Default` in each Parameter**.



### 2. Create a Stack from your template

Go to the [AWS Console for CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active).

Click **Create Stack**:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-1.png)

**Enter a name for your Stack**, and then **Upload a template to Amazon S3**:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-2.png)

Click the **Choose File** button and select your new Template (from where you saved it in Step 1) and **click Next**.

**Verify the Parameter values** to be what you entered into Defaults in your Template.

Using the example values above, it would look like this:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-3.png)

**Click Next**.

**Leave the Options as they are.** Just click **Next**.

If everything looks correct, **click Create**.

After a few minutes your new Stack should go from `CREATE_IN_PROGRESS` to `CREATE_COMPLETE`. If needed, click the Refresh button.

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/cloudformation-4.png)




### 3. View your new CloudWatch Alarms

You can now view the results of your new Stack of CloudWatch Alarms.

Go to the AWS Console for CloudWatch: <br>
https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#alarm:alarmFilter=ANY

The name you gave as the `SiteName` will preface each new CloudWatch Alarm name. 





# Configuring Nagios

For this example, we will be manually configuring one Nagios **Host** and one Nagios **Service**.

As of 2014-12-11 the HUIT production Nagios host is `palantir2.unix.fas.harvard.edu` (`140.247.35.189`).

The Nagios config files are located in `/usr/local/nagios/etc`



### 1. Create Nagios Host objects

**Edit the file `hosts.cfg` and add a new `define host` stanza.**

This is an example Nagios Host object. This defines the RDS Instance with the example values we used previously.
You can copy & paste this text block if you like:
```
define host {
	use					aws-host-active-check
	host_name			aws.test.huit.harvard.edu:hdjhee21ma2vq6
	_AWS_Data			aws.test:AWS/RDS:DBInstanceIdentifier
	contact_groups		aws-dev-group
}
```
(The attributes `use` and `contact_groups` are defined elsewhere in Nagios configurations. They are outside the 
scope of this tutorial.)

In this example the unique values for our Stack are:
- `aws.test.huit.harvard.edu` as the first part of `host_name` - **this is the value from the `SiteName` Parameter, with dashes changed to dots**

- `hdjhee21ma2vq6` as the second part of `host_name` - **this is the `MasterDB` Parameter from above**

- `aws.test` as the first part of `_AWS_Data` (the part - **this is a short form of `SiteName` which is used in Nagios displays**

**These three unique values must be taken from the `SiteName` and `MasterDB` from your stack.**

When done, **save the file.**




### 2. Create Nagios Service objects

**Edit the `services.cfg` and add a new `define service` stanza.**

This is an example Nagios Service object. This defines a specific CloudWatch Alarm for the RDS Instance from the example above.
Again, you can copy & paste this text block if you like.
```
define service {
	use					aws-service-CloudFront-Alarm
	host_name			aws.test.huit.harvard.edu:hdjhee21ma2vq6
	service_description	ReadIOPS
	contact_groups		aws-dev-group
	check_command		check_AWS_CloudWatch_Alarm!cloudhacks
}
```
**Note that the `host_name` is the same as the `host_name` defined in the Host object in the previous step.**

Here also, the attributes `use`, `contact_groups`, and `check_command` are configured elsewhere in Nagios, and are 
outside the scope of thie documentation.

The `service_description` is `ReadIOPS` which is one of the CloudWatch Alarm Metrics defined in your Template. 

(Optional: If you want to see it, search your Template for `"MetricName": "ReadIOPS"` and you'll see the specific CloudWatch Alarm being defined under `Resources`.)

When done, **save the file.**




### 3. Test the Nagios configuration

We now have one new Nagios Host and one new Nagios Service defined. 

**On the Nagios server as root, run `service nagios configtest | grep -v "WARNING: Extinfo objects are deprecated"`**

(The `grep` is to remove warnings from Nagios version 2 config entries which need to get cleaned up but aren't fatal.)

As long as you see the following all-clear, you're all set:
```
Total Warnings: 0
Total Errors:   0
```





### 4. Reload the Nagios configuration

Perform a configuration reload by running as root: `service nagios reload`

You can now go to https://nagios.huit.harvard.edu/nagios/ and search for your new Host:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/nagios-1.png)

(You don't need to fill in the full Host name. A partial name will return the alphabetically first partial match.)






# Testing CloudWatch Alarms and Nagios

The AWS CLI can be used to change the state of a CloudWatch Alarm to create a notification test. 
The command is `aws cloudwatch set-alarm-state` with options described below.

The AWS CLI tools are installed on the Nagios server.

    % which aws
    /usr/bin/aws

**Important**: In order to use the AWS CLI, user credentials need to be created. Doing so is outside the scope of this 
docuemnt, but details can be found at https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/tree/master/Configuration

For this example we will use `sudo` to run the commands as the `nagios` user which has AWS CLI credentials. 

**Each admin who uses the AWS CLI should create their own private credentials.**

Using the example configurations from above, we need to supply the AWS CLI with the following parameters:

1. `--alarm-name` (which is the name in CloudWatch not Nagios)
2. `--state-reason` - some text message explaining that it's a test
3. `--state-value` - can be `ALARM` or `OK` or `INSUFFICIENT_DATA`
4. `--profile` for the particular VPC, to get the AWS CLI credentials

A complete test command line for the example CloudWatch Alarm and Nagios Host and Service created above looks like:
```
# sudo -u nagios aws cloudwatch set-alarm-state --alarm-name "aws-test-huit-harvard-edu RDS Read IO" --state-reason "Nagios TEST ONLY - please disregard this test - setting AWS alarm to ALARM" --state-value ALARM --profile cloudhacks
```

You can verify that the Alarm was set by going to CloudWatch and selecting that Alarm, then view the **History** tab:

![](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/Documentation/Images/nagios-2.png)

Note that the Alarm changed to state ALARM from our CLI command, but since the actual state is OK it flipped right back. 
This ensures that manual twiddling of an Alarm state for testing purposes won't make that Alarm become out-of-sync 
with the actual condition being monitored.





# Attribution

This documentation produced November 2014 by Stefan Wuensch <stefan_wuensch@harvard.edu>

Assistance and additional material by Stephen Martino <stephen_martino@harvard.edu>


