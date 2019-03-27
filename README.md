# Data-Analytics-using-AWS
Data Analytics Using Cloud

Introduction 
Amazon Kinesis Analytics is the easiest way to process streaming data in real time with standard SQL without having to learn new programming languages or processing frameworks. Amazon Kinesis Analytics enables you to create and run SQL queries on streaming data so that you can gain actionable insights and respond to your business and customer needs promptly. 
This tutorial walks you through the process of ingesting streaming log data, aggregating that data, and persisting the aggregated data so that it can be analyzed and visualized. You will create a complete end-to-end system that integrates several AWS services. You will analyze a live stream of Apache access log data and aggregate the total request for each HTTP response type every minute. To visualize this data in near real-time, you will use a user interface (UI) tool that will chart the results. 
Architecture 
One of the major benefits to using Amazon Kinesis Analytics is that an entire analysis infrastructure can be created with a serverless architecture. The system created in this tutorial will implement Amazon Kinesis Firehose, Amazon Kinesis Analytics, and Amazon Elasticsearch Service (Amazon ES). Each of these services is designed for seamless integration with one another. The architecture is depicted below. 
 

The web server in this example will be an Amazon Elastic Compute Cloud (EC2) instance. You will install the Amazon Kinesis Agent on this Linux instance, and the agent will continuously forward log records to an Amazon Kinesis Firehose delivery stream (step 1). Amazon Kinesis Firehose will write each log record to Amazon Simple Storage Service (Amazon S3) for durable storage of the raw log data (step 2), and the Amazon Kinesis Analytics application will continuously run an SQL statement against the streaming input data (step 2). The Amazon Kinesis Analytics application will create an aggregated data set every minute and output that data to a second Firehose delivery stream (step 3). This Firehose delivery stream will write the aggregated data to an Amazon ES domain (step 4). Finally, you will create a view of the streaming data using Kibana to visualize the output of your system (step 5). 
Estimate Your Costs 
The total cost of analyzing your Apache access logs will vary depending on several things: how many web log records you ingest, the complexity of your Amazon Kinesis Analytics SQL queries, and the instance size, storage choice, and redundancy chosen for the Amazon ES domain. This tutorial also creates an EC2 instance to generate a sample Apache access log. The instance size you choose 
and the amount of time that the instance is running will affect the cost. 
Using the default configuration recommended in this guide, it will cost approximately $0.51 to complete the tutorial. This estimate assumes that the infrastructure you create during the tutorial is running for 1 hour. A breakdown of the services used and their associated costs is provided in the following section. 
Services Used and Costs 
AWS pricing is based on your usage of each individual service. The total combined usage of each service will create your monthly bill. For this tutorial, you will be charged for the use of Amazon EC2, Amazon Kinesis Firehose, Amazon S3, Amazon Kinesis Analytics, and Amazon ES. 
Amazon EC2 
Description: Amazon EC2 provides the virtual application servers, known as instances, to run your web application on the platform you choose. EC2 allows you to configure and scale your compute capacity easily to meet changing requirements and demand. It is integrated into Amazon’s computing environment, allowing you to leverage the AWS suite of services. 
How Pricing Works: Amazon EC2 pricing is based on four components: the instance type you choose (EC2 comes in 40+ types of instances with options optimized for compute, memory, storage, and more), the region your instances are based in, the software you run, and the pricing model you select (on-demand instances, reserved capacity, spot, etc.). For more information, see Amazon EC2 pricing.1 
Example: Assume your log files reside on a single Linux t2.nano EC2 instance in the US East region. With an on-demand pricing model, the monthly charge for your virtual machine will be $4.68. For this tutorial, assuming that the log generating instance runs for 1 hour, your EC2 cost is estimated to be $0.0065 [= ($4.68 per month / 30 days per month / 24 hours per day) * 1 hour]. 
Amazon Kinesis Firehose 
Description: Amazon Kinesis Firehose is a fully managed service for delivering real-time streaming data to destinations such as Amazon S3, Amazon Redshift, or Amazon ES. With Firehose, you do not need to write any applications or manage any resources. You configure your data producers to send data to Firehose and it automatically delivers the data to the destination that you specified. 

How Pricing Works: Amazon Kinesis Firehose pricing is based on the volume of data ingested into Amazon Kinesis Firehose, which is calculated as the number of data records you send to the service, times the size of each record, rounded up to the nearest 5 KB. For example, if your data records are 42 KB each, Amazon Kinesis Firehose will count each record as 45 KB of data ingested. In the US East region, the price for Amazon Kinesis Firehose is $0.035 per GB of data ingested. For more information, see Amazon Kinesis Firehose Pricing.2 
Example: In this tutorial, you will create two separate Amazon Kinesis Firehose delivery streams. One will receive the data from your Apache access log producer, and the other will receive the output from an Amazon Kinesis Analytics application. Assume the producer sends 500 records per second, and that each record is less than 5 KB in size (typical for an Apache access log record). The monthly estimate for data ingestion into the Firehose delivery stream in this example would be: 
•	•  The price in the US East region is $0.035 per GB of data ingested. 
•	•  Record size, rounded up to the nearest 5 KB = 5 KB 
•	•  Data ingested (GB per sec) = (500 records/sec * 5 KB/record) / 1,048,576 KB/GB = 0.002384 GB/sec 
•	•  Data ingested (GB per month) = 30 days/month * 86,400 sec/day * 0.002384 GB/sec = 6,179.81 GB/month 
•	•  Monthly charge: 6,179.81 * $0.035/GB = $216.29 
For this tutorial, assume that the system is only ingesting data for 1 hour. The cost specifically for this tutorial would be approximately $0.30 [= ($216.29 per month / 30 days per month / 24 hours per day) * 1 hour]. 
The second Firehose delivery stream is receiving records at a much less frequent rate. Because the Amazon Kinesis Analytics application is outputting only a few rows of data every minute, the cost for that delivery stream is correspondingly smaller. Assuming only five records per minute are ingested, and each record is less than 5 KB, the cost for the delivery stream is $0.00005 for the 1-hour duration assumed for this tutorial. 
Amazon S3 
Description: Amazon S3 provides secure, durable, and highly-scalable cloud storage for the objects that make up your application. Examples of objects you can store include source code, logs, images, videos, and other artifacts that are created when you deploy your application. Amazon S3 makes it is easy to use object storage with a simple web interface to store and retrieve your files from anywhere on the web, meaning that your website will be reliably available to your visitors. 
How Pricing Works: Amazon S3 pricing is based on five components: the type of Amazon S3 storage you use, the region you store your WordPress content (e.g., US East vs. Asia Pacific - Sydney), the amount of data you store, the number of requests you or your users make to store new content or retrieve the content, and the amount of data that is transferred from Amazon S3 to you or your users. For more information, see Amazon S3 Pricing.3 
Example: Using Standard Storage in the US East region, if you store 5 GB of content, you would pay $0.15 per month. If you created your account in the past 12 months, and you are eligible for the AWS Free Tier,4 you would pay $0.00 per month. For this tutorial, assume that the producer creates 5 GB of data. Over a 1- hour period, the total cost for storing the records in Amazon S3 is $0.0002 [= ($0.15 per month / 30 days per month / 24 hours per day) * 1 hour]. 
Amazon Kinesis Analytics 
Description: Amazon Kinesis Analytics is the easiest way to process and analyze streaming data in real-time with ANSI standard SQL. It enables you to read data from Amazon Kinesis Streams and Amazon Kinesis Firehose, and build stream processing queries that filter, transform, and aggregate the data as it arrives. Amazon Kinesis Analytics automatically recognizes standard data formats, parses the data, and suggests a schema, which you can edit using the interactive schema editor. It provides an interactive SQL editor and stream processing templates so you can write sophisticated stream processing queries in just minutes. Amazon Kinesis Analytics runs your queries continuously, and writes the processed results to output destinations such as Amazon Kinesis Streams and Amazon Kinesis Firehose, which can deliver the data to Amazon S3, Amazon Redshift, and Amazon ES. Amazon Kinesis Analytics automatically provisions, deploys, and scales the resources required to run your queries. 
How Pricing Works: With Amazon Kinesis Analytics, you pay only for what you use. You are charged an hourly rate based on the average number of Kinesis Processing Units (KPUs) used to run your stream processing application. 
A single KPU is a unit of stream processing capacity comprised of 4 GB memory, 1 vCPU compute, and corresponding networking capabilities. As the complexity of your queries varies, and the demands on memory and compute vary in response, Amazon Kinesis Analytics will automatically and elastically scale the number of KPUs required to complete your analysis. There are no resources to provision and no upfront costs or minimum fees associated with Amazon Kinesis Analytics. For more information, see Amazon Kinesis Analytics Pricing.5 
Example: This example assumes that the system is running for 1 hour in the US East region. The SQL query in this tutorial is very basic and will not consume more than one KPU. Given that the price for Amazon Kinesis Analytics in US East is $0.11 per KPU-hour, and the tutorial runs for 1 hour, the total cost for the usage of Amazon Kinesis Analytics is $0.11. 
Amazon Elasticsearch Service 
Description: Amazon ES is a popular open-source search and analytics engine for big data use cases such as log and click stream analysis. Amazon ES manages the capacity, scaling, patching, and administration of Elasticsearch clusters for you while giving you direct access to the Elasticsearch API. 
How Pricing Works: With Amazon ES, you pay only for what you use. There are no minimum fees or upfront commitments. You are charged for Amazon Elasticsearch instance hours, an Amazon Elastic Block Store (EBS) volume (if you choose this option), and standard data transfer fees. For more information, see Amazon Elasticsearch Service Pricing.6 
Example: For this tutorial, assuming the defaults are chosen when you create the Amazon ES domain, the total cost can be calculated as follows: 
An instance type of m3.medium.elasticsearch costs $0.094 per hour * 1 hour = $0.094. 



Steps
1.	Step 1: Set Up Prerequisites 
Before you begin analyzing your Apache access logs with Amazon Kinesis Analytics, make sure you complete these prerequisites: 
o	•  Create an AWS Account 
o	•  Start an EC2 Instance 
o	•  Prepare Your Log Files 
This tutorial assumes that the AWS resources have been created in the US East AWS region (us-east-1). 
Create an AWS Account 
If you already have an AWS account, you can skip this prerequisite and use your existing account. To create AWS account if you do not already one: 
1.	Go to http://aws.amazon.com/. 
2.	Choose Create an AWS Account. 
3.	Follow the online instructions. 
Part of the sign-up procedure involves receiving a phone call and entering a PIN using the phone keypad. 
Start an EC2 Instance 
The steps outlined in this tutorial assume that you are using an EC2 instance as the web server and log producer. You can use an existing EC2 instance launched from Amazon Linux AMI (version 2015.09 or later) or Red Hat Enterprise Linux (version 7 or later). Otherwise, you can follow the guide Getting Started with Amazon EC2 Linux Instances to start a new instance.7 If you create a new instance, choose Amazon Linux AMI for your operating system. An instance type of t2.micro is sufficient for this tutorial. 
You also want to ensure that your EC2 instance has an AWS Identity and Access Management (IAM) role configured with permission to write to Amazon Kinesis Firehose and Amazon CloudWatch. For more information, see IAM Roles for Amazon EC2.8 
Once you have launched the EC2 instance, you will need to connect to it via SSH. To connect to your instance, follow the guide Connect to Your Linux Instance.9 
Prepare Your Log Files 
Because Amazon Kinesis Analytics can analyze your streaming data in near real- time, this tutorial is much more effective when a live stream of Apache access log data is used. If your EC2 instance is not serving HTTP traffic, you will need to generate continuous sample log files. 
To create a continuous stream of log file data on your EC2 instance, download, install, and run the Fake Apache Log Generator from Github.10 Follow the instructions on the project page and configure the script for infinite log file generation. 

Step 2: Create an Amazon Kinesis Firehose Delivery Stream 
In Step 1, you created log files on your web server. Before they can be analyzed with Amazon Kinesis Analytics (Step 6), the log data must first be loaded into AWS. Amazon Kinesis Firehose is a fully managed service for delivering real-time streaming data to destinations such as Amazon Simple Storage Service (Amazon S3), Amazon Redshift, or Amazon Elasticsearch Service (Amazon ES). 
In this step, you will create an Amazon Kinesis Firehose delivery stream to save each log entry in Amazon S3 and to provide the log data to the Amazon Kinesis Analytics application that you will create later in this tutorial. 
To create the Amazon Kinesis Firehose delivery stream: 
1.	Open the Amazon Kinesis console at 
https://console.aws.amazon.com/kinesis. 
2.	Click Go to Firehose. 
3.	Click Create Delivery Stream. 
4.	On the Destination screen: 
1.	For Destination, choose Amazon S3. 
2.	For Delivery stream name, enter web-log-ingestion-stream. 
3.	For S3 bucket, choose CreateNew S3 Bucket. 
You will be prompted to provide additional information for the new bucket: 
For Bucket name, use a unique name. You will not need to use the name elsewhere in this tutorial. However, Amazon S3 bucket names are required to be globally unique. 
For Region, choose US Standard. Choose Create Bucket. 
4.	For S3 prefix, leave it blank (default). 
5.	Click Next. 
5. On the Configuration screen (see below), you can leave all fields set to their default values. However, you will need to choose an IAM role so that Amazon Kinesis Firehose can write to your Amazon S3 bucket on your behalf. 
1.	For IAM role, choose Create/Update Existing IAM Role. 
2.	Click Next. 

Step 3: Install and Configure the Amazon Kinesis Agent 
Now that you have an Amazon Kinesis Firehose delivery stream ready to ingest your data, you can configure the EC2 instance to send the data using the Amazon Kinesis Agent software. The agent is a stand-alone Java software application that offers an easy way to collect and send data to Firehose. The agent continuously monitors a set of files and sends new data to your delivery stream. It handles file rotation, checkpointing, and retry upon failures. It delivers all of your data in a reliable, timely, and simple manner. It also emits Amazon CloudWatch metrics to help you better monitor and troubleshoot the streaming process. 
The Amazon Kinesis Agent can pre-process records from monitored files before sending them to your delivery stream. It has native support for Apache access log files, which you created in Step 1. When configured, the agent will parse log files in the Apache Common Log format and convert each line in the file to JSON format before sending to your Firehose delivery stream, which you created in Step 2. 
1. To install the agent, see Download and Install the Agent.11 
2. For detailed instructions on how to configure the agent to process and send log data to your Amazon Kinesis Firehose delivery stream, see Configure and Start the Agent.12 
To configure the agent for this tutorial, modify the configuration file located at /etc/aws-kinesis/agent.json using the template below. Replace full-path-to-log-file with a file pattern that represents the path to your log files and a wildcard if you have multiple log files with the same naming convention. For example, it might look similar to: “/var/log/httpd-access.log*”. The value will be different, depending on your use case. Replace name-of-delivery-stream with the name of the Firehose delivery stream you created in Step 2. 

{
"cloudwatch.endpoint": "monitoring.us-east-1.amazonaws.com", "cloudwatch.emitMetrics": true,
"firehose.endpoint": "firehose.us-east-1.amazonaws.com", "flows": [ 
{
"filePattern": "full-path-to-log-file", "deliveryStream": "name-of-delivery-stream", "dataProcessingOptions": [
{ 
}] } 
] } 
"initialPostion": "START_OF_FILE", "maxBufferAgeMillis":"2000", "optionName": "LOGTOJSON", "logFormat": "COMBINEDAPACHELOG" 
3.	Start the agent manually by issuing the following command: 
sudo service aws-kinesis-agent start


Once started, the agent will look for files in the configured location and send the records to the Firehose delivery stream.

