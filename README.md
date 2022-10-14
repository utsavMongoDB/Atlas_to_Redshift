# Operational Analytics with Atlas and Redshift


## Introduction

The modern business world demands expedited decision-making, improved customer experience, and increased productivity. Gone are those days when business intelligence relied only on past data through batch processing. 
The order of the day is Operational analytics, which relies on measuring the existing or real-time operations of the business along with its past data trends.

## Why Operational Analytics?
First and foremost there is an exponential increase in data volumes and their varieties. Traditional DW needs to evolve constantly to meet this demand of changing needs.
Most recent data are no more tabular in nature. Databases evolved into JSONs, Social logs, Images, Videos, and Time Series format.

Of late the legacy relational database models are becoming showstoppers for programming and advanced analytics. With the legacy ways of datastore, the performance becomes a big bottleneck as the data grows into terabytes and petabytes.

So the need of the hour is to have a cohesive data model, which takes care of both the day-to-day operational data and its past.

Thus the coexistence of Atlas and Redshift evolves as the perfect solution for the business need.

## Integration Framework

The data from MongoDB Atlas is synced with Amazon Redshift in a two-step approach.

### Step 1: One-Time Load
MongoDB Atlas has direct connectors with Apache Spark. Using the spark connectors the data is migrated from MongoDB Atlas to Redshift as a one-time load.

### Step2: (Near) Real-Time Data Sync
With the help of the MongoDB Atlas triggers or Amazon MSK, any delta changes to the database can be continuously written to S3 bucket. From the S3 bucket, data can be loaded into the Redshift either through scheduled AWS Glue jobs or can be accessed as an external table.

In this demonstration, we provided step by step approach for each of these scenarios.


## Pre-requisite: 
a) Good understanding of AWS Redshift, Glue, and S3 services.

b) Good understanding of MongoDB Atlas and Application services.

c) VPC and Network settings are already set up as per the security standards.

d) Redshift Database.

e) S3 bucket to store the JSON files.

f) MongoDB Atlas cluster [for free cluster creation](https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/)

g) Tools: [VSCode](https://code.visualstudio.com/), [MongoDB Compass](https://www.mongodb.com/products/compass), [Docker](https://www.docker.com/)





## One-Time Load

### Architecture diagram

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/01.One-Time%20Data%20Load.png)

### Step by Step Instructions

a. Create a [MongoDB Atlas cluster](https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster).

b. Configure the MongoDB Atlas cluster [network security](https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/) and [access](https://www.mongodb.com/docs/atlas/tutorial/create-mongodb-user-for-cluster/).

c. Load the sample [customer_activity](https://github.com/mongodb-partners/Atlas_to_Redshift/blob/main/code/data/customer_activity.json) data to a collection using [MongoDB Compass](https://www.mongodb.com/docs/compass/current/import-export/).

d. Create a [Amazon Redshift Cluster ](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-launch-sample-cluster.html).

e. Configure the Amazon Redshift Cluster [network security](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) and [access](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-connect-to-cluster.html).

f. Note down the username and password.

g. [Create an AWS role](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions_create-policies.html#:~:text=the%20following%20procedure.-,To%20create,-a%20role%20for) with AmazonDMSRedshiftS3Role and AWSGlueServiceRole policies and note down the role name.

h. Create an AWS Glue connection with the Amazon Redshift Database.


Select "Connector" from the left side menu of AWS Glue Studio. Click "Create Connection" to create a new connection. 



![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/GlueConnection1.png)



Provide a name for the connection, and select "Amazone Redshift" for the connection type, the Redshift credential created in the last step.

Note: The redshift connection name is hardcoded in the python script. Please note down the connection name, if you are giving it differently.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/GlueConnection2.png)


i. Create a glue job in AWS Glue studio.


j. Select the Spark script editor and click "Create".

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob1.png)


k. Copy the code from the [link](https://github.com/mongodb-partners/Atlas_to_Redshift/blob/main/code/glue_pyspark_atlas_to_redshift.py) to the "script" tab. Overwrite if there is a template code available already.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob2.png)



l. Update the connection details for MongoDB URI and Database credentials. The Redshift connection and Redshift database name are hardcode. Update if required to your values.



m. Configure the job name and AWS role in the "Job details" tab. You can keep all the other parameters as default.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/gluejob3.png)



n. Save the job and click "Run"

o. Ensure the job ran successfully 

p. Validate the table and data in Redshift.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/redshiftoutput.png)


##  (Near) Real-Time Data Sync 


The Change Data Capture(CDC) feature of MongoDB Atlas is utilized to capture real-time data. Migrate the real-time data to the S3 bucket and then to Redshift by **ANY ONE ** of the following methods.


#### with Amazon Managed Streaming for Apache Kafka (Amazon MSK)

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/MSKArchitecture.png)


#### With Glue: 

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/12.AWS%20Glue%20s3tocatalog%20Connections%204.png)


#### with Redshift Spectrum (External Table)

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/11.AWS%20Glue%20s3tocatalog%20Connections%204.png)


### Step by Step Instructions for setting up Amazon MSK Job

a. Create an [MSK Cluster](https://docs.aws.amazon.com/msk/latest/developerguide/create-cluster.html)

b. Create  [custom plugins](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect-plugins.html) for MongoDB Atlas(source) using the [zip file](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/Scripts/Mongo-Kafka-connector.zip)

c. Create [custom plugins](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect-plugins.html) for S3(sink) using the [zip file](https://www.confluent.io/hub/confluentinc/kafka-connect-s3#:~:text=will%20be%20run.-,Download,-By%20downloading%20you)

d. Create a [source connector](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect-connectors.html#:~:text=MSK%20Connect.-,Creating,-a%20connector) to MongoDB Atlas using the custom plugin and [code](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/Scripts/atlassourceconnector.txt)

e. Create a [sink connector](https://docs.aws.amazon.com/msk/latest/developerguide/msk-connect-connectors.html#:~:text=MSK%20Connect.-,Creating,-a%20connector) to the S3 bucket using the custom plugin and [code](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/Scripts/s3sinkconnector.txt).

f. Insert the data to MongoDB Atlas collection and ensure the data are written to the S3 bucket.


### Step by Step Instructions for setting up Glue Studio Job
1. The data from MongoDB Atlas can be continuously written to S3 bucket using the Data Federation and MongoDB Atlas triggers. 
 Please refer to the [link](https://www.mongodb.com/developer/products/atlas/automated-continuous-data-copying-from-mongodb-to-s3/) for the step-by-step instructions to capture the data to S3.

 For any further reference, please follow the MongoDB documentation [link](https://www.mongodb.com/docs/atlas/data-federation/config/config-aws-s3/)

2. Create a AWS Glue job to move the data from S3 bucket to AWS Redshift
      
a. Create the Glue Connections Redshift Database.

Navigate to the AWS Glue console and to the "Data Catalog" menu on the left panel. Select "Connections" and Click on "Add Connection".  Enter the parameters taking  guidance of the screenshots attached. 

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/05.AWS%20Glue%20Redshift%20Connections%201.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/06.AWS%20Glue%20Redshift%20Connections%202.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/07.AWS%20Glue%20Redshift%20Connections%203.png)

b.Create the Glue Connection for S3 bucket.
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/10.AWS%20Glue%20s3tocatalog%20Connections%201.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/10.AWS%20Glue%20s3tocatalog%20Connections%202.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/10.AWS%20Glue%20s3tocatalog%20Connections%203.png)

Test these connections are working fine. 

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/10.AWS%20Glue%20s3tocatalog%20Connections%204.png)

C.Create the Crawler to populate Database and Tables in AWS Glue Catalog from S3.
Navigate to "Crawlers" menu on the left side panel and click "Add Crawler". Add all the required information for the crawler taking guidance from the attached screenshots.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%201.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%202.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%203.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%204.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%205.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%206.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%207.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%208.png)

Once the crawlers are created successfully , run the crawler and ensure its successful completion.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%2010.png)

Click the "Tables" menu from the left side and ensure the required tables are created. 

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/08.AWS%20Glue%20Redshift%20Crawler%209.png)

c.Now we are all set to create the Job to map and populate the redshift tables. 
  Click " Jobs" from the left side menu and select "Visual with a source and target"
  
  select S3 bucket as source and Redshift as target from the dropdown.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob4.png)


update the source configuration

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob5.png)


Validate the mapping and alter as required.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob6.png)

update the destination configuration

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob7.png)

The scripts for the conversions are created automatically.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob9.png)

update the job details tab with job name and role. all other parameters are kept as default.

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/Gluejob8.png)

Run the job and ensure it's successful completion. Use the logs and Error logs generated for debugging. (if required)

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/09.AWS%20Glue%20Job%2008.png)
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/09.AWS%20Glue%20Job%2009.png)

Verify the data is populated successfully in the Redshift table.
![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/09.AWS%20Glue%20Job%2010.png)

### Step by Step Instruction for setting up Redshift Spectrum - External Table

Redshift Spectrum host the S3 bucket data as an external table. Provided the reference and steps to create the extenal table in the following link

[Redshift Specturm - external table](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/Scripts/external_tables_in_redshift.sql)

Reference [link](https://docs.aws.amazon.com/redshift/latest/dg/tutorial-query-nested-data.html) for querying JSON structure data in Redshift.


## Analytical Services using Redshift ML.

The data thus populated from MongoDB Atlas either through AWS Glue or as a external tables in Redshift can be utilized to train the models . Redshift ML services enables to directly use the Sagemaker Models to train and infer results.

Please refer the [link](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/Scripts/RedshiftML_for_CustomerActivity.sql) for training a model and infer result from the model.



## Business Use Cases:
Enterprises like Retail, Banking & Finance and Manufacturing are in great demand for the operational analytics for it's various real-time analytics.

A few are captured below

![](https://github.com/Babusrinivasan76/atlastoredshift/blob/main/images/04.Key%20Business%20Use%20Cases.png)

## Summary: 
With the synergy it creates by having Atlas for its operational efficiency and Redshift for its DWH excellence, all the “Operational Analytics” use cases can be delivered in no time. The solution can be extended to integrate the AI/ML needs using the AWS SageMaker.

Hope you are able to setup the integration successfully. For any further reference pls reach out to partners@mongodb.com


