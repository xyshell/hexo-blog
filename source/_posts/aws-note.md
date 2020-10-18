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

## Install

### AWS CLI and Boto3
#### Amazon Linux 2

The AWS CLI is already installed on Amazon Linux 2.

Install Python 3:

```bash
sudo yum install -y python3-pip python3 python3-setuptools
```

Install Boto3:

```bash
pip3 install boto3 --user
```

#### macOS

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


### Docker

```bash
sudo amazon-linux-extras install docker
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

```bash
aws configure
aws sts get-caller-identity
```

## S3

Upload folder to s3:
```bash
aws s3 cp <path-to-source-folder> s3://<path-to-target-folder> --recursive --exclude ".DS_Store"
```

## Dynamodb

create table:
```bash
aws dynamodb create-table \
    --table-name Music \
    --key-schema AttributeName=Artist,KeyType=HASH \
                 AttributeName=SongTitle,KeyType=RANGE \    
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \    
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5
```

describe table:
```bash
aws dynamodb describe-table --table-name Music
```

put item:
```bash
aws dynamodb put-item \
    --table-name Music \
    --item '{
        "Artist": {"S": "Dream Theater"},
        "AlbumTitle": {"S": "Images and Words"},        
        "SongTitle": {"S": "Under a Glass Moon"} }'
```

scan table:
```bash
aws dynamodb scan --table-name Music
```

# Boto3 - EC2

This section follows https://github.com/linuxacademy/content-dynamodb-deepdive

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

## Backing Up EC2 Instances

![Backing-Up-EC2-Instances](/photos/aws-note/Backing-Up-EC2-Instances.png)

### Create-Backups

```python

from datetime import datetime

import boto3


def lambda_handler(event, context):

    ec2_client = boto3.client('ec2')
    regions = [region['RegionName']
               for region in ec2_client.describe_regions()['Regions']]

    for region in regions:

        print('Instances in EC2 Region {0}:'.format(region))
        ec2 = boto3.resource('ec2', region_name=region)

        instances = ec2.instances.filter(
            Filters=[
                {'Name': 'tag:backup', 'Values': ['true']}
            ]
        )

        # ISO 8601 timestamp, i.e. 2019-01-31T14:01:58
        timestamp = datetime.utcnow().replace(microsecond=0).isoformat()

        for i in instances.all():
            for v in i.volumes.all():

                desc = 'Backup of {0}, volume {1}, created {2}'.format(
                    i.id, v.id, timestamp)
                print(desc)

                snapshot = v.create_snapshot(Description=desc)

                print("Created snapshot:", snapshot.id)
```

### Prune-Backups

```python
import boto3


def lambda_handler(event, context):

    account_id = boto3.client('sts').get_caller_identity().get('Account')
    ec2 = boto3.client('ec2')
    regions = [region['RegionName']
               for region in ec2.describe_regions()['Regions']]

    for region in regions:
        print("Region:", region)
        ec2 = boto3.client('ec2', region_name=region)
        response = ec2.describe_snapshots(OwnerIds=[account_id])
        snapshots = response["Snapshots"]

        # Sort snapshots by date ascending
        snapshots.sort(key=lambda x: x["StartTime"])

        # Remove snapshots we want to keep (i.e. 3 most recent)
        snapshots = snapshots[:-3]

        for snapshot in snapshots:
            id = snapshot['SnapshotId']
            try: # EBS might be using this snapshot
                print("Deleting snapshot:", id)
                ec2.delete_snapshot(SnapshotId=id)
            except Exception as e:
                print("Snapshot {} in use, skipping.".format(id))
                continue
```

### Removing Unattached EBS Volumes

```python
import boto3


def lambda_handler(object, context):

    # Get list of regions
    ec2_client = boto3.client('ec2')
    regions = [region['RegionName']
               for region in ec2_client.describe_regions()['Regions']]

    for region in regions:
        ec2 = boto3.resource('ec2', region_name=region)
        print("Region:", region)

        # List only unattached volumes ('available' vs. 'in-use')
        volumes = ec2.volumes.filter(
            Filters=[{'Name': 'status', 'Values': ['available']}])

        for volume in volumes:
            v = ec2.Volume(volume.id)
            print("Deleting EBS volume: {}, Size: {} GiB".format(v.id, v.size))
            v.delete()
```

### Deregistering Old AMIs

```python
import datetime
from dateutil.parser import parse

import boto3


def days_old(date):
    parsed = parse(date).replace(tzinfo=None)
    diff = datetime.datetime.now() - parsed
    return diff.days


def lambda_handler(event, context):

    # Get list of regions
    ec2_client = boto3.client('ec2')
    regions = [region['RegionName']
               for region in ec2_client.describe_regions()['Regions']]

    for region in regions:
        ec2 = boto3.client('ec2', region_name=region)
        print("Region:", region)

        amis = ec2.describe_images(Owners=['self'])['Images']

        for ami in amis:
            creation_date = ami['CreationDate']
            age_days = days_old(creation_date)
            image_id = ami['ImageId']
            print('ImageId: {}, CreationDate: {} ({} days old)'.format(
                image_id, creation_date, age_days))

            if age_days >= 2:
                print('Deleting ImageId:', image_id)

                # Deregister the AMI
                ec2.deregister_image(ImageId=image_id)
```

# Boto3 - Dynamodb

This chapter follows https://github.com/linuxacademy/content-lambda-boto3

```python
import boto3
client = boto3.client('dynamodb', endpoint_url='http://localhost:8000') # dynamodb-local
client.list_tables()
```

### Create Tables

```python
dynamodb = boto3.resource('dynamodb')

table = dynamodb.create_table(
    TableName='Movies',
    KeySchema=[
        {
            'AttributeName': 'year',
            'KeyType': 'HASH'  # Partition key
        },
        {
            'AttributeName': 'title',
            'KeyType': 'RANGE'  # Sort key
        }
    ],
    AttributeDefinitions=[
        {
            'AttributeName': 'year',
            'AttributeType': 'N'
        },
        {
            'AttributeName': 'title',
            'AttributeType': 'S'
        },

    ],
    ProvisionedThroughput={
        'ReadCapacityUnits': 5,
        'WriteCapacityUnits': 5
    }
)

print('Table status:', table.table_status)

print('Waiting for', table.name, 'to complete creating...')
table.meta.client.get_waiter('table_exists').wait(TableName='Movies')
print('Table status:', dynamodb.Table('Movies').table_status)
```

### Load Data

```python
dynamodb = boto3.resource('dynamodb')

table = dynamodb.Table('Movies')

with open("moviedata.json") as json_file:
    movies = json.load(json_file, parse_float=decimal.Decimal)
    for movie in movies:
        year = int(movie['year'])
        title = movie['title']
        info = movie['info']

        print("Adding movie:", year, title)

        table.put_item(
            Item={
                'year': year,
                'title': title,
                'info': info,
            }
        )
```

`moviedata.json`:
```json
[{
    "year": 2013,
    "title": "Rush",
    "info": {
      "directors": ["Ron Howard"],
      "release_date": "2013-09-02T00:00:00Z",
      "rating": 8.3,
      "genres": [
        "Action",
        "Biography",
        "Drama",
        "Sport"
      ],
      "image_url": "http://ia.media-imdb.com/images/M/MV5BMTQyMDE0MTY0OV5BMl5BanBnXkFtZTcwMjI2OTI0OQ@@._V1_SX400_.jpg",
      "plot": "A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda.",
      "rank": 2,
      "running_time_secs": 7380,
      "actors": [
        "Daniel Bruhl",
        "Chris Hemsworth",
        "Olivia Wilde"
      ]
    }
  },
]
```

### Put Item

```python

class DecimalEncoder(json.JSONEncoder):
    '''Helper class to convert a DynamoDB item to JSON'''

    def default(self, o):
        if isinstance(o, decimal.Decimal):
            if abs(o) % 1 > 0:
                return float(o)
            else:
                return int(o)
        return super(DecimalEncoder, self).default(o)

title = "The Big New Movie"
year = 2015

response = table.put_item(
    Item={
        'year': year,
        'title': title,
        'info': {
            'plot': "Nothing happens at all.",
            'rating': decimal.Decimal(0)
        }
    }
)
print("PutItem succeeded:")
print(json.dumps(response, indent=4, cls=DecimalEncoder))

```

### Get Item

```python
from botocore.exceptions import ClientError

title = "The Big New Movie"
year = 2015

try:
    response = table.get_item(
        Key={
            'year': year,
            'title': title
        }
    )
except ClientError as e:
    print(e.response['Error']['Message'])
else:
    item = response['Item']
    print("GetItem succeeded:")
    print(json.dumps(item, indent=4, cls=DecimalEncoder))
```

### Update Item

```python
title = "The Big New Movie"
year = 2015

response = table.update_item(
    Key={
        'year': year,
        'title': title
    },
    UpdateExpression="set info.rating = :r, info.plot=:p, info.actors=:a",
    ExpressionAttributeValues={
        ':r': decimal.Decimal(5.5),
        ':p': "Everything happens all at once.",
        ':a': ["Larry", "Moe", "Curly"]
    },
    ReturnValues="UPDATED_NEW"
)

print("UpdateItem succeeded:")
print(json.dumps(response, indent=4, cls=DecimalEncoder))

```

```python
title = "The Big New Movie"
year = 2015

response = table.update_item(
    Key={
        'year': year,
        'title': title
    },
    UpdateExpression="set info.rating = info.rating + :val",
    ExpressionAttributeValues={
        ':val': decimal.Decimal(1)
    },
    ReturnValues="UPDATED_NEW"
)

print("UpdateItem succeeded:")
print(json.dumps(response, indent=4, cls=DecimalEncoder))

```

```python
title = "The Big New Movie"
year = 2015

# Conditional update (will fail)
print("Attempting conditional update...")

try:
    response = table.update_item(
        Key={
            'year': year,
            'title': title
        },
        UpdateExpression="remove info.actors[0]",
        ConditionExpression="size(info.actors) >= :num",
        ExpressionAttributeValues={
            ':num': 3
        },
        ReturnValues="UPDATED_NEW"
    )
except ClientError as e:
    if e.response['Error']['Code'] == "ConditionalCheckFailedException":
        print(e.response['Error']['Message'])
    else:
        raise
else:
    print("UpdateItem succeeded:")
    print(json.dumps(response, indent=4, cls=DecimalEncoder))

```

### Delete Item

```python
title = "The Big New Movie"
year = 2015

print("Attempting a conditional delete...")

try:
    response = table.delete_item(
        Key={
            'year': year,
            'title': title
        }
    )
except ClientError as e:
    if e.response['Error']['Code'] == "ConditionalCheckFailedException":
        print(e.response['Error']['Message'])
    else:
        raise
else:
    print("DeleteItem succeeded:")
    print(json.dumps(response, indent=4, cls=DecimalEncoder))
```

### Query

```python
from boto3.dynamodb.conditions import Key

print("Movies from 1985")

response = table.query(
    KeyConditionExpression=Key('year').eq(1985)
)

for i in response['Items']:
    print(i['year'], ":", i['title'])
```

```python
print("Movies from 1992 - titles A-L, with genres and lead actor")

response = table.query(
    ProjectionExpression="#yr, title, info.genres, info.actors[0]",
    # Expression Attribute Names for Projection Expression only.
    ExpressionAttributeNames={"#yr": "year"},
    KeyConditionExpression=Key('year').eq(
        1992) & Key('title').between('A', 'L')
)

for i in response[u'Items']:
    print(json.dumps(i, cls=DecimalEncoder))
```

### Scan

```python
fe = Key('year').between(1950, 1959)
pe = "#yr, title, info.rating"
# Expression Attribute Names for Projection Expression only.
ean = {"#yr": "year", }
esk = None

response = table.scan(
    FilterExpression=fe,
    ProjectionExpression=pe,
    ExpressionAttributeNames=ean
)

for i in response['Items']:
    print(json.dumps(i, cls=DecimalEncoder))

while 'LastEvaluatedKey' in response:
    response = table.scan(
        ProjectionExpression=pe,
        FilterExpression=fe,
        ExpressionAttributeNames=ean,
        ExclusiveStartKey=response['LastEvaluatedKey']
    )

    for i in response['Items']:
        print(json.dumps(i, cls=DecimalEncoder))
```

### Delete table

```python
import boto3

dynamodb = boto3.resource('dynamodb')

table = dynamodb.Table('Movies')

table.delete()
```

# Boto3 - S3

This chapter follows https://github.com/linuxacademy/content-lambda-boto3

## Resizing Images

![Resizing-Images](/photos/aws-note/Resizing-Images.png)

```python
import os
import tempfile

import boto3
from PIL import Image

s3 = boto3.client('s3')
DEST_BUCKET = os.environ['DEST_BUCKET']
SIZE = 128, 128


def lambda_handler(event, context):

    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        thumb = 'thumb-' + key
        with tempfile.TemporaryDirectory() as tmpdir:
            download_path = os.path.join(tmpdir, key)
            upload_path = os.path.join(tmpdir, thumb)
            s3.download_file(source_bucket, key, download_path)
            generate_thumbnail(download_path, upload_path)
            s3.upload_file(upload_path, DEST_BUCKET, thumb)

        print('Thumbnail image saved at {}/{}'.format(DEST_BUCKET, thumb))


def generate_thumbnail(source_path, dest_path):
    print('Generating thumbnail from:', source_path)
    with Image.open(source_path) as image:
        image.thumbnail(SIZE)
        image.save(dest_path)
```

To get pillow pkg, find it at https://pypi.org/project/Pillow/

```bash
unzip Pillow-5.4.1-cp37-cp37m-manylinux1_x86_64.whl
```

```bash
rm -rf Pillow-5.4.1.dist-info
```

```bash
zip -r9 lambda.zip lambda_function.py PIL
```

upload the zip file to AWS Lambda.

## Importing CSV Files into DynamoDB

![Importing-CSV-Files-into-DynamoDB](/photos/aws-note/Importing-CSV-Files-into-DynamoDB.png)

```python
import csv
import os
import tempfile

import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Movies')
s3 = boto3.client('s3')


def lambda_handler(event, context):

    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        with tempfile.TemporaryDirectory() as tmpdir:
            download_path = os.path.join(tmpdir, key)
            s3.download_file(source_bucket, key, download_path)
            items = read_csv(download_path)

            with table.batch_writer() as batch:
                for item in items:
                    batch.put_item(Item=item)


def read_csv(file):
    items = []
    with open(file) as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            data = {}
            data['Meta'] = {}
            data['Year'] = int(row['Year'])
            data['Title'] = row['Title'] or None
            data['Meta']['Length'] = int(row['Length'] or 0)
            data['Meta']['Subject'] = row['Subject'] or None
            data['Meta']['Actor'] = row['Actor'] or None
            data['Meta']['Actress'] = row['Actress'] or None
            data['Meta']['Director'] = row['Director'] or None
            data['Meta']['Popularity'] = row['Popularity'] or None
            data['Meta']['Awards'] = row['Awards'] == 'Yes'
            data['Meta']['Image'] = row['Image'] or None
            data['Meta'] = {k: v for k,
                            v in data['Meta'].items() if v is not None}
            items.append(data)
    return items

```

## Transcribing Audio

![Transcribing-Audio](/photos/aws-note/Transcribing-Audio.png)

TranscribeAudio:
```python
import boto3

s3 = boto3.client('s3')
transcribe = boto3.client('transcribe')


def lambda_handler(event, context):

    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        object_url = "https://s3.amazonaws.com/{0}/{1}".format(
            source_bucket, key)
        response = transcribe.start_transcription_job(
            TranscriptionJobName='MyTranscriptionJob',
            Media={'MediaFileUri': object_url},
            MediaFormat='mp3',
            LanguageCode='en-US'
        )
        print(response)
```

ParseTranscription:
```python
import json
import os
import urllib.request

import boto3


BUCKET_NAME = os.environ['BUCKET_NAME']

s3 = boto3.resource('s3')
transcribe = boto3.client('transcribe')


def lambda_handler(event, context):
    job_name = event['detail']['TranscriptionJobName']
    job = transcribe.get_transcription_job(TranscriptionJobName=job_name)
    uri = job['TranscriptionJob']['Transcript']['TranscriptFileUri']
    print(uri)

    content = urllib.request.urlopen(uri).read().decode('UTF-8')

    print(json.dumps(content))

    data = json.loads(content)

    text = data['results']['transcripts'][0]['transcript']

    object = s3.Object(BUCKET_NAME, job_name + '-asrOutput.txt')
    object.put(Body=text)
```

## Detecting Faces with Rekognition

![Detecting-Faces-with-Rekognition](/photos/aws-note/Detecting-Faces-with-Rekognition.png)

```python
import os

import boto3

TABLE_NAME = os.environ['TABLE_NAME']

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(TABLE_NAME)
s3 = boto3.resource('s3')
rekognition = boto3.client('rekognition')


def lambda_handler(event, context):

    # Get the object from the event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    obj = s3.Object(bucket, key)
    image = obj.get()['Body'].read()
    print('Recognizing celebrities...')
    response = rekognition.recognize_celebrities(Image={'Bytes': image})

    names = []

    for celebrity in response['CelebrityFaces']:
        name = celebrity['Name']
        print('Name: ' + name)
        names.append(name)

    print(names)

    print('Saving face data to DynamoDB table:', TABLE_NAME)
    response = table.put_item(
        Item={
            'key': key,
            'names': names,
        }
    )
    print(response)
```

