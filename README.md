# HUIT Guide to configuring AWS Monitoring and Notifications

This documentation covers the creation of monitoring of Amazon Web Services using 
[AWS CloudWatch Alarms](http://aws.amazon.com/documentation/cloudwatch/) and 
[Nagios](http://nagios.sourceforge.net/docs/nagioscore/4/en/toc.html). 

HUIT recommends AWS applications be monitored with a two-factor approach: 
- *CloudWatch Alarms* which are Amazon-provided monitoring services within the Virtual Private Cloud (inside looking in) and 
- *Nagios Service Checks* which monitor from outside the AWS environment (outside looking in)



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
  


## External Documentation

### PDFs
[CloudFormation User Guide PDF](http://awsdocs.s3.amazonaws.com/AWSCloudFormation/latest/cfn-ug.pdf)

### HTML
[CloudFormation Templates](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html)


## Attribution

This documentation produced November 2014 by Stefan Wuensch <stefan_wuensch@harvard.edu>

Assistance and additional material by Stephen Martino <stephen_martino@harvard.edu>
