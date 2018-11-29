> This document details the setup steps for the integration of Deep Security and AWS Security Hub. For the full context of the integration and other important information, please [refer to the README](README.md).

# Setting Up The Integration With Deep Security

Setting up the integration involves 4 steps;

1. Configure Deep Security To Send Events To Amazon SNS
2. Configure AWS Security Hub
3. Configure The AWS Lambda Function
4. Configuring AWS IAM Permissions

## 1. Configure Deep Security To Send Events To Amazon SNS

The configuration to send all Deep Security events to Amazon SNS is very simple. It's [detailed here](https://help.deepsecurity.trendmicro.com/sns.html) in the Deep Security Help Center. In lieu of copying those steps here, please refer to the Help Center, that will always be up to date.

By default, this configuration sends all Deep Security events to the specified Amazon SNS topic. You can [filter what events are sent](https://help.deepsecurity.trendmicro.com/Events-Alerts/sns-json-config.html?Highlight=sns) using a simple JSON policy language (very similar to AWS IAM).

This integration only uses a subset of Deep Security events. Essentially only sending critical and high severity events to AWS Security Hub by default. This class of events is more closely related to the core concept of an AWS Security Hub finding.

**Caution:** if you use the Deep Security event policy language to prevent relevant events from being sent to an Amazon SNS topic, those events won't show up in the AWS Security Hub. This is unlikely to happen but something to be aware of if you're filtering the event stream outbound from your Deep Security installation.

## 2. Configure AWS Security Hub

AWS Security Hub is available as an open preview. Simply access the service from the AWS Management Console and click "Enable Security Hub".

![Enable AWS Security Hub](docs/enable-security-hub.png)

This will walk you through the initial process of setting up the required permissions and structures to support the AWS Security Hub. 

Once that initial step is complete, you need to subscribe to Trend Micro's Deep Security in order to permit the service to receive events from your Deep Security installation.

![Subscribe to Trend Micro:Deep Security](docs/subscribe-to-deep-security.png)

## 3. Configure The AWS Lambda Function

Using the CloudFormation template in this repository, you can easily deploy the required AWS Lambda function and assign the proper permissions via the AWS IAM execution role.

The code requires the following permissions to run;

- read access to the target Amazon SNS topic that Deep Security is sending events to
- write access to the AWS Security Hub API, specifically the ImportFindings function calls

## 4. Configuring AWS IAM Permissions

Deep Security allows you to setup multiple AWS accounts under one installation. This means that events information from multiple AWS accounts may be flowing to one centralized AWS Security Hub.

This is typically the desired result.

**But** in order to prevent spoofing of findings, AWS Security Hub restricts source accounts. By default, you cannot send events from account B to the AWS Security Hub in account A.

In order to enable this functionality, the integration supports assuming an IAM role in account B when called from account A.

That creates a work flow of;

- account A hosts the integration function for AWS Lambda
- account A's function receives an event flagged as account B
- the integration function assumes a role in account B
- the integration function then sends the event as a finding from account B to the AWS Security Hub in account A

In order to simplify the creation of this role, we've [provided an Amazon CloudFormation template](cf-deep-security-aff-forward-to-aws-security-hub.yaml) in the repo. This template should be run in account B with the parameter of *TargetHubAccountId* set to account A.