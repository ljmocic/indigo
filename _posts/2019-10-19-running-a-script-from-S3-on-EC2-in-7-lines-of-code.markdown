---
title: "Running a script from S3 on EC2 with 7 lines of code"
layout: post
date: 2019-10-19 12:48
image: https://cdn-images-1.medium.com/max/2000/1*OSxo-IvxdoGFXZLFQwQ0mg.png
headerImage: true
tag:
- s3
- ec2
- script
category: blog
author: ljmocic
description: Use the power of EC2 instance to execute a resource-hungry script
---

## **Problem**

The other day I was approached by a client who had a 2011 MacBook Air. He was trying to execute some computationally intensive script which only required Python to be installed. His script took longer than 30 minutes to execute on his machine.

He wanted to place his script in S3 bucket and simply execute it in the cloud without worrying about anything else.

## **Solution**

The first solution that came to me was the usage of AWS Lambda. But Lambda execution time limit was 5 minutes. Recently it got upgraded to 15 minutes, but it still may not be enough.

The other solution was a simple usage of EC2. But the problem with classic EC2 is connecting and worrying about the instance. If the instance gets left behind it can pile up the bills and we do not want it to happen.

What we need:

* EC2 instance

* Python preinstalled

* Execute a script on startup

* Terminate after finishing execution of the script

The final solution was to use Amazon Linux Image for EC2 instance which has the capability of receiving startup commands for execution and has Python built in.

## **Environment**

In order to follow this guide, please install [Python](https://www.python.org/downloads/).

Also, I use [Visual Studio Code](https://code.visualstudio.com/), it is my favorite editor. But you can use whatever you want.

We will need the boto3 package which will help us control AWS resources.

In order for to make dealing with AWS account easier, install aws-cli.

    pip install boto3 && pip install aws-cli

Login to your AWS account. You can create an account on [aws.amazon.com](https://aws.amazon.com/).

    aws configure

Enter the access key and secret key from AWS IAM and make sure that you have permissions to view the S3 bucket and create EC2 instances.

## **Code**

Create main.py and open Visual Studio Code.

    touch main.py && code .

Import boto3 library.

    import boto3

We will need to use the S3 and EC2 so we create clients for them.

    EC2 = boto3.client('ec2')

    S3 = boto3.client('s3')

So now we need to download the script from S3, the first argument is the ***bucket*** which has the script. Second is the ***path*** of the script in the bucket and the third one is the ***download path*** in your local system.

    S3.download_file('test-bucket', 's3-script.py', 'local-script.py')

The next steps:

* Open the file in reading mode.

* Load the script into the string with appending all of the lines with ‘\n’ newline.

* Tell the EC2 to run our script by specifying the code in *UserData* argument.

* Use *ImageId* which has the option to pass *UserData*.

* Seet *m3.medium* size of the AMI.

* Run only on 1 instance: *MinCount=1* and *MaxCount=1*.

* Terminate: *InstanceInitiatedShutdownBehavior=’terminate’*

    with open('local-script.py', 'r') as f;
       script = '\n'.join(f)
       instance = EC2.run_instances(ImageId='ami-00b94673edfccb7ca',
                                    InstanceType='m3.medium',
                                    MinCount=1,
                                    MaxCount=1, 
                        InstanceInitiatedShutdownBehavior='terminate', 
                        UserData=script)

And that is it. You can run it.

    python main.py

It will execute your script from S3 on EC2 instance.

You can find the working sample [here](https://medium.com/media/8cfbdb80d03e22ce5839b04a57ff2e37).
