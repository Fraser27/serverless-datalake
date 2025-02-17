
# Serverless Datalake with Amazon EventBridge

### Architecture
![architecture](https://user-images.githubusercontent.com/25897220/210151716-5e670221-db78-4681-b711-1ef8d94ea102.png)

<br>
<br>

### Overview
This solution demonstrates how we could build a serverless ETL solution with Amazon Eventbridge, Kinesis Firehose, AWS Glue.

* Amazon Eventbridge is a high throughput serverless event router used for ingesting data, it provides out-of-the-box integration with multiple AWS services. We could generate custom events from multiple hetrogenous sources and route it to any of the AWS services without the need to write any integration logic. With content based routing , a single event bus can route events to different services based on the match/un-matched event content.

* Kinesis Firehose is used to buffer the events and store them as json files in S3.

* AWS Glue is a serverless data integration service. We've used Glue to transform the raw event data and also to generate a 
schema using AWS Glue crawler. AWS Glue Jobs run some basic transformations on the incoming raw data. It then compresses, partitions the data and stores it in parquet(columnar) storage. AWS Glue job is configured to run every hour. A successful AWS Glue job completion would trigger the AWS Glue Crawler that in turn would either create a schema on the S3 data or update the schema and other metadata such as newly added/deleted table partitions.

* Glue Workflow
<img width="1033" alt="glue-workflow" src="https://user-images.githubusercontent.com/25897220/210151743-399ad0cf-9cfd-47c4-b193-6bf899074a41.png">

* Glue Crawler
<img width="1099" alt="glue-crawler" src="https://user-images.githubusercontent.com/25897220/210151749-c84a909a-df2a-468c-a5c6-37532cb3ce6b.png">

* Datalake Table
<img width="1106" alt="glue-table" src="https://user-images.githubusercontent.com/25897220/210151755-048092c5-a53b-4e3f-a46b-2900ab3f8470.png">

* Once the table is created we could query it through Athena and visualize it using Quicksight. <br>
Athena Query
<img width="1364" alt="athena_query" src="https://user-images.githubusercontent.com/25897220/210151765-0c80f4dc-d909-4597-a5f3-4d4b4897ee97.png">

* This solution also provides a test lambda that generates 500 random events to push to the bus. The lambda is part of the **test_stack.py** nested stack. The lambda should be invoked on demand.
Test lambda name: **serverless-event-simulator-dev**



This solution could be used to build a datalake for API usage tracking, State Change Notifications, Content based routing and much more.


### Setup

1.  Note: You may need to do a cdk bootstrap if you haven't used cdk before. Link: https://docs.aws.amazon.com/cdk/v2/guide/bootstrapping.html

```
e.g. cdk bootstrap aws://ACCOUNT-NUMBER-1/REGION-1 aws://ACCOUNT-NUMBER-2/REGION-2 ...

cdk bootstrap aws://{your-account-id}/{your-region}
```
<br>

2. install the latest cdk cli on your local machine:
```
npm install -g aws-cdk 
```

3. create a python virtualenv:
```
$ python3 -m venv .venv
```

4. After the init process completes and the virtualenv is created, you can use the following
step to activate your virtualenv.

```
$ source .venv/bin/activate
```

*  If you are a Windows platform, you would activate the virtualenv like this:

```
% .venv\Scripts\activate.bat
```

5. Once the virtualenv is activated, you can install the required dependencies.

```
$ pip install -r requirements.txt
```

6. At this point you can now synthesize the CloudFormation template for this code.

```
$ cdk synth -c environment_name=dev
```

7. You can deploy the generated CloudFormation template using the below command
```
$ cdk deploy -c environment_name=dev ServerlessDatalakeStack
```

8. Configurations for dev environment are defined in cdk.json. S3 bucket name is created on the fly based on account_id and region in which the cdk is deployed

9. After you've successfully deployed this stack on your account, you could test it out by executing the test lambda thats deployed as part of this stack.
Test lambda name: **serverless-event-simulator-dev** . This lambda will push 500 random events to the event-bus.
