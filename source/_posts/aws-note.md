---
title: AWS Note
date: 2020-06-08 01:30:49
tags: aws
toc: true
---

# AWS Basics

## AWS website

[infrastructure.aws](https://infrastructure.aws/)

[TCO(total cost of ownership) Calculator](http://awstcocalculator.com/)

[Pricing Calculator](https://calculator.aws/#/)

## Install the AWS CLI and Boto3

### Amazon Linux 2

The AWS CLI is already installed on Amazon Linux 2.

Install Python 3:

```bash
sudo yum install -y python3-pip python3 python3-setuptools
```

Install Boto3:

```bash
pip3 install boto3 --user
```

### macOS

Install Python3 using Homebrew:

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Install Python 3:

```bash
brew install python
```

Insert the Homebrew Python directory at the top of your PATH environment variable:

```bash
export PATH="/usr/local/opt/python/libexec/bin:$PATH"
```

Verify you are using Python 3:

```bash
python --version
```

Install the AWS CLI and Boto3:

```bash
pip install awscli boto3 --upgrade --user
```

## Configuring your AWS environment

Obtain your AWS access key and secret access key from the AWS Management Console. Run the following command:

```bash
aws configure
```

This sets up a text file that the AWS CLI and Boto3 libraries look at by default for your credentials: `~/.aws/credentials`.

The file should look like this:

```txt
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

## Test Your Credentials

AWS CLI
Run the following command:

```bash
aws sts get-caller-identity
```

The output should look like this:

```python
{
    "UserId": "AIDAJKLMNOPQRSTUVWXYZ",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:userdevuser"
}
```

# AWS CLI

## Upload folder to S3

```bash
aws s3 cp <path-to-source-folder> s3://<path-to-target-folder> --recursive --exclude ".DS_Store"
```

# Boto3

## HelloWorld

### List all S3 bucket name:

```python
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

### Spin up an ec2 instance

```python
import boto3
ec2 = boto3.client('ec2')
response = ec2.run_instances(
    ImageId='ami-0947d2ba12ee1ff75', # Amazon Linux 2 AMI (HVM), SSD Volume Type
    InstanceType='t2.micro',
    KeyName='xyshell',
    MinCount=1,
    MaxCount=1,
    SubnetId='subnet-05b78e3323bcabddc'
)
print(response['Instances'][0]['InstanceId'])
```

## Stopping EC2 Instances Nightly

![Stopping-EC2-Instances-Nightly.png](/photos/aws-note/Stopping-EC2-Instances-Nightly.png)

```python
# lambda_function.py
import boto3

def lambda_handler(event, context):
    ec2_client = boto3.client('ec2')
    # get list of regions
    regions = [region['RegionName'] for region in ec2_client.describe_regions()['Regions']]
    # iterate over each region
    for region in regions:
        ec2 = boto3.resource('ec2', region_name=region)
        print('Region:', region)
        # get only running instances
        instances = ec2.instances.filter(
            Filters=[
                {'Name': 'instance-state-name',
                 'Values': ['running']}
            ]
        )
        # stop instances
        for instance in instances:
            instance.stop()
            print('stopped instance:', instance.id)
```