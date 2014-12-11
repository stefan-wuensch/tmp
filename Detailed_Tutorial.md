This folder contains documentation & instructions on how to set up / configure / use / maintain the monitoring services.

**Note**: The `Old_Examples` subdirectory is included here as an archive of old documentation from Summer 2014. 
Those docs are obsoleted by the Nagios configuration auto-generation built by Stefan during Fall 2014. 


# HUIT Guide to configuring AWS Monitoring and Notifications

This documentation covers the creation of monitoring of Amazon Web Services using 
[AWS CloudWatch Alarms](http://aws.amazon.com/documentation/cloudwatch/) and 
[Nagios](http://nagios.sourceforge.net/docs/nagioscore/4/en/toc.html). 

HUIT recommends AWS applications be monitored with a two-factor approach: 
- **CloudWatch Alarms** which are Amazon-provided monitoring services within the Virtual Private Cloud (inside looking in) and 
- **Nagios Service Checks** which monitor from outside the AWS environment (outside looking in)

CloudWatch Alarms are used to monitor aspects (Metrics) of AWS services which are not part of the public-facing application. 
For example, CPU Utilization metrics of a database instance are not part of the customer experience when viewing a web site. 
Nagios checks are used to monitor both the customer experience (web pages, content) but also to receive and process the CloudWatch 
notifications. This allows Nagios to be a single point of aggregation and distribution of alert messages. 

HUIT Nagios is able to send [alert notifications](http://nagios.sourceforge.net/docs/nagioscore/4/en/notifications.html) to a variety 
of destinations, including (but not limited to):
- HUIT SOC Operations Center staffed 24x7 (via ITO / OpenView)
- SMS (phones / pagers) via SMTP
- SMS via Cellular Modem
- email (SMTP)
This documentation will focus on notifications to SOC Operations via OpenView.



## Overview

**CloudFormation Templates** are [JSON](http://en.wikipedia.org/wiki/JSON) files which define the configuration of 
**CloudFormation Stacks**. Stacks are collections of AWS resources which are related to a common application or function.
For more information see http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html

Two Stacks are needed at minimum for each HUIT customer requiring AWS monitoring:

1. a Stack of SNS Topics which define the target of CloudWatch Alarms' messages
2. a Stack of CloudWatch Alarms which define the Metrics to be monitored, and define the thresholds, time periods, and actions

For each Stack of CloudWatch Alarms, a set of Nagios configuration objects will be built by scripts on the Nagios server. 
Nagios configuration objects will also be built for EC2 Instances (if desired) automatically by additional scripts on the Nagios server.
See [the folder of scripts in Github](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/tree/master/Code-to-connect-AWS-and-Nagios) 
for the specifics.


## Important notes

- This guide assumes a working knowledge of AWS, specifically the following:
  - [AWS Console](https://console.aws.amazon.com/)
  - [CloudWatch Alarms](https://console.aws.amazon.com/cloudwatch/) and [Alarm Metrics](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#metrics:)
  - [CloudFormation](https://console.aws.amazon.com/cloudformation/) and [CloudFormation Templates](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html)
  - [Simple Notification Service (SNS)](https://console.aws.amazon.com/sns/), SNS Subscriptions, and SNS Topics

- This guide also assumes a general familiarity with Nagios concepts such as:
  - [Types of Nagios objects](http://nagios.sourceforge.net/docs/nagioscore/4/en/configobject.html)
  - [Service Checks](http://nagios.sourceforge.net/docs/nagioscore/4/en/servicechecks.html)
  - [Monitoring Publicly Available Services](http://nagios.sourceforge.net/docs/nagioscore/4/en/monitoring-publicservices.html)
  - [Event Handlers](http://nagios.sourceforge.net/docs/nagioscore/4/en/eventhandlers.html)
  - [Active Checks](http://nagios.sourceforge.net/docs/nagioscore/4/en/activechecks.html) vs. [Passive Checks](http://nagios.sourceforge.net/docs/nagioscore/4/en/passivechecks.html)
  
- This guide does not document the use of [Github](https://github.com/HUIT-Systems-Management-Linux-UNIX) for storage of CloudFormation Templates, 
nor Nagios configuration files nor scripts. However, the use of Github for centralized storage is assumed.


## Getting Started with AWS

### Start with an existing CloudFormation Template as a guide

See the [folder of templates](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/tree/master/JSON-Templates) that are live in production for 
Harvard Publishing and Communications (HPAC) and Harvard Web Publishing (HWP).

Specifically, the template 
[online-learning-harvard-edu.json](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/blob/master/JSON-Templates/online-learning-harvard-edu.json) 
is one of the simplest. It only contains Alarm configurations for the following types and quantities of AWS resources:

- One ELB (Elastic Load Balancer)
- One RDS (Relational Database Service)
- One EC2 Instance for a Drupal admin node
- Two EC2 Instances for Drupal webserver nodes

This sample template `online-learning-harvard-edu.json` provides the following **13** CloudWatch Alarm metrics. 
Each Alarm is created by being defined in the `Resources` section of the template.

- EC2 Admin Node `CPUUtilization` as `AdminCPUAlarm`
- ELB `HealthyHostCount` (absolute minimum) as `ELBHealthyHostLessThanOne`
- ELB `HealthyHostCount` (based on Auto Scaling Group minimum) as `ELBHealthyHost`
- ELB `RequestCount` as `ELBRequestCount`
- ELB `Latency` as `ELBLatencyAlarm`
- ELB `HTTPCode_ELB_5XX` as `ELB5XXAlarm`
- ELB `HTTPCode_ELB_4XX` as `ELB4XXAlarm`
- RDS `CPUUtilization` as `RDSCPUHighAlarm`
- RDS `FreeStorageSpace` as `RDSFreeStorageSpaceAlarm`
- RDS `ReadIOPS` as `RDSReadIOPSHighAlarm`
- RDS `WriteIOPS` as `RDSWriteIOPSHigh`
- RDS `ReadLatency` as `RDSreadLatency`
- RDS `WriteLatency` as `RDSwriteLatency`

All of the above CloudWatch Metrics (the left-hand attribute) are documented by Amazon online. A Google search for `aws cloudwatch` 
plus the name of the resource type and the Metric should take you right to that documentation.

For example: The ELB Metric `HTTPCode_ELB_4XX` is described (along with all the other ELB Metrics) at 
http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/US_MonitoringLoadBalancerWithCW.html 
which can be found by searching Google for `aws cloudwatch ELB HTTPCode_ELB_4XX`

Another example: The RDS Metric `WriteIOPS` is documented at 
http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/rds-metricscollected.html 
which can be found by searching Google for `aws cloudwatch RDS WriteIOPS`


**Make a copy of that template `online-learning-harvard-edu.json`. We will use it as the basis for creating a new Stack of CloudWatch Alarms.**


### Parameters

Note that the `Default` values given in the template are only for convenience and elminiating errors. 

However, each time a CloudFormation Stack is created or updated there must be values entered for each of the Parameters. 
If the `Default` values in the template are kept current, then creating / updaing the Stack requires very little effort. 
It is **highly** recommended that not only are Parameter `Default` values be used, but that they be updated in the template 
as soon as possible after any termination / re-creation of Resources in the stack being monitored.

The following **eight** Parameters are the only updates needed to your copy of the example template, *assuming that the above list of CloudWatch Alarm Metrics are sufficient*.
(Creation of CloudWatch Alarms using Metrics other than those already found in the sample `online-learning-harvard-edu.json` is outside the scope of this documentation.)

**Also note** that if the customer has provisioned their site Stack with *either* creation Parameters or **Outputs** which contain some or all of the following eight 
CloudWatch Stack Parameters, or if Stack Resources show usable resource IDs, you may be able to simply view those values from the 
[AWS Console for CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active) 
in order to fill in the following Parameters. 

Example of using Outputs to find Alarm Stack Parameters: 
In the production HWP [HPAC online-learning.harvard.edu stack](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active&stackId=arn:aws:cloudformation:us-east-1:014311208322:stack%2FHPACLearnProd%2Fe619b710-1d58-11e4-b2d5-50e241629418&tab=outputs) 
there is an Output `ElasticLoadBalancerName` as well as `RDSInstanceName`. These are the two values needed for Step 1 and Step 4 respectively.

Example of using Resources to find Alarm Stack Parameters:
The production HWP [HPAC online-learning.harvard.edu](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active&stackId=arn:aws:cloudformation:us-east-1:014311208322:stack%2FHPACLearnProd%2Fe619b710-1d58-11e4-b2d5-50e241629418&tab=resources) 
shows a "Logical ID" `DBInstance` with "Physical ID" `hdxmrpsfnzbr5v`. This is the same as the Output named `RDSInstanceName`, and either of these is what's needed in Step 4.

The two examples above illustrate that you *might* be able to find needed values for the Alarms Stack by simply viewing the customer site stack. However, 
since that it not guaranteed to be the case, the below instructions reference the more specific AWS Console views.


#### Filling in the Default values

1. **ElasticLoadBalancer** is the Parameter for the name of the ELB.
Look in the [AWS Console for Load Balancers](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LoadBalancers:) and find the full name 
of the ELB in the customer stack you are to monitor. Fill it in here as the Default.
Example: `HPACLearn-ElasticL-JXIRIRNJ5LR6`

2. **AdminNode** is the Parameter for the name of the Drupal admin node.
In the [AWS Console for EC2 Instances](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:sort=instanceId) find the Instance that is the Admin Node for 
this customer. Enter the **Instance ID** as the Default value in the template.
Example: `i-f4458015`

3. **AutoScalingGroupMinSize** is the Parameter for minimum number of EC2 web server instances allowed. (We are assuming there will be only one web **admin** node.)
This value is used to look up thresholds in a Mapping for use in the ELB Healthy Host Count alarm. 
Search the template for `HealthyHostCountMapByGroupMinimum` if you want to see the mapping of AutoScalingGroupMinSize to the thresholds. 
Optionally see also the [AWS documentation on CloudFormation Mappings in templates](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html).

  To find the correct value for this Parameter, go to the [AWS Console for EC2 Auto Scaling Groups](https://console.aws.amazon.com/ec2/autoscaling/home?region=us-east-1#AutoScalingGroups:view=details). 
  Find the Auto Scaling Group for the customer site, and note the **Min** value for that Group. That is the value to be entered in the template for `AutoScalingGroupMinSize` default.

  For example: For the HPAC / HWP site `online-learning.harvard.edu`, the name of the prod Auto Scaling Group is `HPACLearnProd-WebServerGroup-I274WYW1N1MO` and it has a Min value 
  of **2**. When the CloudWatch Alarms Stack is created in CloudFormation, the `AutoScalingGroupMinSize` value of 2 will be used to determine values for `EvaluationPeriods`, `ComparisonOperator`, 
  `ComparisonText`, and `Threshold`. 

  Of course, using a Mapping in a template is **completely optional** but by doing so it allows much greater flexibility and scalability. It makes one Parameter value able to 
  populate (in this example) **four** different Resource attributes.


4. **MasterDB** is the Parameter for the name of the RDS (database) Instance. Go to the [AWS Console for RDS Instances](https://console.aws.amazon.com/rds/home?region=us-east-1#dbinstances:) 
and find the RDS Instance for this customer's application. Fill in the name of the Instance as this Default. Example: `hdxmrpsfnzbr5v`

5. **CriticalAlarmTopic** is the name of the SNS Topic which will send alarm notifications to Nagios. This topic name needs to be looked up in 
the [AWS Console for SNS](https://console.aws.amazon.com/sns/) after the Topics Stack has been created. 
Example: `arn:aws:sns:us-east-1:014311208322:HUIT_Nagios_Critical` 

  **_TO DO_** Find out if any AWS VPC can talk to an existing SNS Topic in another VPC!! If it can, we don't have to create a Topic Stack for every customer!!

6. **DBClass** is the type and size of RDS. The `AllowedValues` should already include all popular types of RDS, but if not simply add to that list. 
(For a full list of current Instance Classes, see http://aws.amazon.com/rds/details/ or for previous generations see http://aws.amazon.com/rds/previous-generation/) 
Then from the same [AWS Console for RDS Instances](https://console.aws.amazon.com/rds/home?region=us-east-1#dbinstances:) as in Step 4 find the RDS to be monitored and select (click) it. 
The value shown for `Instance Class` in the Console is the value to be used for `DBClass` in the template Default.
Example: `db.m1.small`

7. **DBName** is the database name itself. Unlike `MasterDB` in step 4 above (which is the name of the *RDS Instance*) the `DBName` is the 
application-layer name of the database. It's this name that database clients use as well. Again refer to the same 
[AWS Console for RDS Instances](https://console.aws.amazon.com/rds/home?region=us-east-1#dbinstances:) as in Step 4 and Step 6. Find the `DB Name` and 
use this for the Default in the template. 
Example: `drupaldb`

8. **SiteName** is the Parameter for the name of the public-facing customer website. This value is used to construct Nagios objects. 
It must be unique across all AWS VPCs and sites that are monitored by HUIT Nagios. 
The format is: website name as FQDN, with dots (periods) replaced by dashes (hyphens). 
(AWS does not allow dots in Alarm Names, and the value of `SiteName` in the template is used to construct Alarm Names.) 
Examples:

Public Site Name		| SiteName in CloudFormation Template
------------------------------- | ------------------------------
`harvard.edu` 			| `harvard-edu`
`news.harvard.edu` 		| `news-harvard-edu`
`campaign.harvard.edu` 		| `campaign-harvard-edu`
`online-learning.harvard.edu` 	| `online-learning-harvard-edu`



### Create the new CloudWatch Alarms Stack

Now that the new CloudFormation template is complete, creating the Stack of Alarms is very easy!

1. Go to the [AWS Console for CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active).

2. Perform a **Create Stack** operation, and name the new stack as explicitly / specifically as possible.

  For the Template, use the option `Upload a template to Amazon S3` and "Browse" to your newly-created template from above.
  
3. Verify that the Default Parameters are correct, or update as needed.

4. Skip adding any Tags at this time. (Tags can be used to add additional metadata to the Stack. 
See [the docs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-resource-tags.html) for more.)

5. For the final step, verify the Template, Parameters and Options and if all looks good click **Create**. (The default Options are fine.)

The Stack creation will take (typically) between 30 and 90 sec. depending on how many Resources are to be created. 

Once the stack creation is complete, examine the [CloudWatch Alarms Console](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#alarm:alarmFilter=ANY) 
and verify the new Alarms. The names should start with the `SiteName` specified in Step 8 above. 

Typically the **only** Alarms that will show an *INSUFFICIENT_DATA* state are Load Balancer 4XX and 5XX counts (`HTTPCode_ELB_4XX` and/or `HTTPCode_ELB_5XX`). 
Any other *INSUFFICIENT_DATA* state should be examined. There may be an incorrect Parameter given in the template Default or given at the time of stack creation.




## Nagios configuration

Note: For this example, we will be manually configuring one Nagios Host and one Nagios Service. As of this writing (2014-11-30) 
the custom auto-configuration mechanisms need to be updated / fixed to allow for multiple customers with different AWS VPCs.
At the moment the HPAC / HWP sites are the only ones that work with auto-configuration due to some code issues.


### Overview of Nagios configuration

The Nagios "Host" object typically represents a conventional host, such as a server (physical or virtual) or a VIP on a load balancer. 
With AWS, the traditional concept of the "host" is replaced by Instances of various resource types (EC2, RDS, ELB, etc.) so we have to 
change the way we think of a Nagios "Host".

For AWS montitoring with Nagios, the Host object now becomes simply a root object to which we associate Service objects. 

For example: an RDS instance being monitored by CloudWatch isn't a true "host" because we don't know (nor do we care) how 
Amazon is actually hosting the database service. Since it won't matter if a "host" in this model is "up" or "down" - 
and we probably won't be able to tell anyway - we use a special Host `check_command` which **always** returns `OK`.

This is taken from the standard Nagios `checkcommands.cfg` file:
```
define command {
	command_name	FAKE-host-alive
	command_line	$USER1$/check_dummy 0 "Fake Alive Place-Holder"
}
```

That command definition is used by the AWS Host template in Nagios:
```
define host {
	name					aws-host-active-check
	use						generic-host
	contact_groups			aws-critical-group
	check_command			FAKE-host-alive
	register				0
}
```
(Note that Nagios templates are simply a way of allowing related objects to inherit from a common config definition. 
For more on Nagios templates see http://nagios.sourceforge.net/docs/nagioscore/4/en/objectinheritance.html)





### Host Configuration

For this example we will configure a Nagios "Host" which represents an AWS RDS instance. 

A demo Drupal site stack has been set up in advance, and we have already created an Alarm stack template as 
shown above, and created that stack. 

We know from the template (and from looking at the AWS Console for RDS) that the DB Instance Name is 
`cloudwatch-demo-drupal-sitemasterdatabase`. That becomes the basis for the Nagios "Host" name, but it needs more than that.

Nagios Host names **must be unique** across the entire Nagios namespace. Since a particular customer might have 
multiple websites each of which has the same DB Instance Name we need to add something additional to make 
the Host Name unique. 

We simply add the name of the site (which was used in Step 8 of creating the template above) to give us a 
constructed `host_name`:
```
define host {
	use					aws-host-active-check
	host_name			smartino.test.harvard.edu:cloudwatch-demo-drupal-sitemasterdatabase
	_AWS_Data			smartino.test:AWS/RDS:DBInstanceIdentifier
	contact_groups		aws-dev-group
}
```
The `use` parameter inherits from the host template as shown earlier. 

Notes:

1. The `_AWS_Data` parameter is a [custom variable](http://nagios.sourceforge.net/docs/nagioscore/4/en/macros.html) 
which is used by Nagios scripts. Below in the Service example it is explained.
2. The `contact_groups` parameter is overriding the `contact_groups` parameter from the `aws-host-active-check` template. 
Since the template is designed for production use (with the Contact Group `aws-critical-group`) we don't want to 
be sending critical alerts through the prod notification channels for this test example!



### Service Configuration

In this simple example we will show how to create a single Nagios Service. It will be the `ReadIOPS` Metric 
for the RDS Instance defined as the Nagios "Host" above. 

**Important**: Nagios Service objects **must be unique** on a single Nagios Host. In other words, if you want to 
monitor the same Metric more than once for a single AWS resource instance, you **must** give it a different 
Nagios Service name for each one. For example if you wanted to monitor `ReadIOPS` with a certain action at a 
low threshold but also include a different action (like an escalation to other contacts) if that same metric `ReadIOPS` 
goes above a second threshold, you would have to create a different `service_description` for it.

In this example, we are again using a Nagios template to inherit parameters that are common to other services.
(The `aws-service-CloudFront-Alarm` template is quite large so it's not included here. You can view it in the 
`aws-alarms.cfg` file in the [Configuration Examples](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/tree/master/Configuration/Examples).)

Note that the `host_name` is the same as the `host_name` defined in the Host object in the previous step. 
```
define service {
	use					aws-service-CloudFront-Alarm
	host_name			smartino.test.harvard.edu:cloudwatch-demo-drupal-sitemasterdatabase
	service_description	ReadIOPS
	notes				Alarm if hpacdrupaldb ReadIOPs > 100 for 5 minutes (DBInstanceIdentifier = smartino.test.harvard.edu:cloudwatch-demo-drupal-sitemasterdatabase, AlarmName = smartino.test.harvard.edu RDS Read IO, Namespace = AWS/RDS)
	contact_groups		aws-dev-group
	check_command		check_AWS_CloudWatch_Alarm!cloudhacks
}
```
The `notes` are completely optional, but very useful when trying to associate this Nagios Service back to the AWS Cloudwatch Alarm and specific Metric and Threshold.

Again here we are overriding the `contact_groups` from the template because we don't want to send alert notifications to prod channels!

Finally, note that the `check_command` parameter includes the name of the `profile` from the `~nagios/.aws/config` file. This is required as an 
input to the script called by the `check_AWS_CloudWatch_Alarm` command, in order to retrieve the credentials to use when calling the AWS CLI.

This is the Nagios Command definition for `check_AWS_CloudWatch_Alarm`, which can also be found in the config file `checkcommands.cfg` in the 
[directory of config examples](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/tree/master/Configuration/Examples).
```
define command {
	command_name	check_AWS_CloudWatch_Alarm
	command_line	$USER1$/FAS/check_AWS_CloudWatch_Alarm.php --hostName $HOSTNAME$ --hostData $_HOSTAWS_DATA$ --serviceDescription $SERVICEDESC$ --profile $ARG1$
}
```
The Nagios Custom Variable `_AWS_Data` is being used in this Nagios Command definition as the value for the command-line argument `--hostData`. 
(The script `check_AWS_CloudWatch_Alarm.php` can be found in the [directory of scripts](https://github.com/HUIT-Systems-Management-Linux-UNIX/Cloud_Monitoring_Services/tree/master/Code-to-connect-AWS-and-Nagios).)



### Nagios validation and reload

Now that we have our new Host and Service definitions created, the final two steps are to validate the configuration and reload Nagios to use the new config.

1. On the Nagios server as root, run `service nagios configtest`. 

  You can ignore the warnings about "Extinfo objects are deprecated" which are coming from old Nagios version 2 definitions which will be converted to Nagios 4 definitions at a later date. 

  To cut down on the warning output, you could instead run: 
  
  `service nagios configtest | grep -v "WARNING: Extinfo objects are deprecated"`

  As long as you see the following all-clear, you're all set:
  ```
  Total Warnings: 0
  Total Errors:   0
  ```

2. Perform a configuration reload by running as root: 
  
  `service nagios reload`
  
Now you can go to the Nagios UI and see your newly-created Host and Service. 

In this example since the Host Name is `smartino.test.harvard.edu:cloudwatch-demo-drupal-sitemasterdatabase` we can search 
for that in the "**Quick Search:**" box. Go to https://nagios.huit.harvard.edu/nagios/ and enter that Host Name.



## Testing CloudWatch Alarms and Nagios

The AWS CLI can be used to change the state of a CloudWatch Alarm to create a notification test. The command is 
`aws cloudwatch set-alarm-state`

Using the example configurations from above, we need to supply the AWS CLI with the following parameters:

1. `--alarm-name` (which is the name in CloudWatch not Nagios)
2. `--state-reason` - some text message explaining that it's a test
3. `--state-value` - can be `ALARM` or `OK` or `INSUFFICIENT_DATA`
4. `--profile` for the particular VPC, to get the AWS CLI credentials

A complete test command line for the example CloudWatch Alarm and Nagios Host and Service created above looks like:
```
aws cloudwatch set-alarm-state --alarm-name "smartino-test-harvard-edu RDS Read IO" --state-reason "Nagios TEST ONLY - please disregard this test - setting AWS alarm to ALARM" --state-value ALARM --profile cloudhacks
```








## External Documentation

### PDFs
[CloudFormation User Guide PDF](http://awsdocs.s3.amazonaws.com/AWSCloudFormation/latest/cfn-ug.pdf)

### HTML
[CloudFormation Templates](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html)


## Attribution

This documentation produced November 2014 by Stefan Wuensch <stefan_wuensch@harvard.edu>

Assistance and additional material by Stephen Martino <stephen_martino@harvard.edu>
