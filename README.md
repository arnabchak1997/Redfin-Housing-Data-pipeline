# Redfin-Housing-Data-pipeline

**Summary:** <br>
In this project, we will be creating an ETL pipeline for analysing Housing market data and use that to draw visualizations. The dataset will be loaded to S3 from where we will run some transformations on it using Pyspark thorugh an EMR cluster, we will load that transformed data to Snowflake using snowpipe and use it draw insisghts and create visualization using Quicksight.


**Dataset Description:** <br>
The project uses housing market dataset from Redfin Data Centre. Redfin is a real estate brokerage, meaning we have direct access to data from local multiple listing services, as well as insight from our real estate agents across USA. The dataset contains listings of various property types across various cities in different states/county in USA, it also lists the inventory details across states and cities and price details of the properties. Redfin offers housing data at National level, metro level, state level, county level, city level, zip codes, neighbourhood level. As part of this project, we are using the dataset from city level.
<br>

**Tech Stack:** <br>
Languages: SQL, Python (PySpark) <br>
Services: AWS S3, Amazon EMR, EC2, VPC, IAM, Quicksight, Snowflake, Airflow <br>

**Architecture:** <br>

![image](https://github.com/user-attachments/assets/cfbb3d31-e68d-43a9-8840-09c9d565cf7a)

<br>

**Execution Steps:** <br>

**Creation of S3 bukcet:** <br>
As part of this project we will be using a single S3 bukcet to store our raw and transformed data, also we are using the s3 bucket to store the emr cluster logs and data ingestion/transformation scripts. 

![image](https://github.com/user-attachments/assets/2350ee1a-dd76-4349-a6d3-50cd900db445) <br>

**Creation of VPC:** <br>
We are creating a seperate VPC for the EMR cluster which we will be using in the project, we have created a simple VPC with only public subnets. <br>

**Amazon EMR setup** <br>
In order to create the transformation script, we have created an EMR cluster with Spark and Jupyter Enterprise gateway installed. The cluster is having only primary node and core node and for scaling we have used EMR managed scaling with minimum cluster size of 3 instances and maximum of 10 instances. The cluster has been created in a seperate VPC (created in previous step).
We have created an EMR studio and workspace where we have developed the transformation script. The workspace has been attached to the EMR cluster created. 
Note: The cluster has been terminated after the transformation script is created as we have used Airflow to automate the cluster creation, ingestion and data transformation steps in this project. <br>

**Data Transformation:** <br>
We have used Pyspark to apply transformation on the data, the raw city level data has been loaded into the S3 bucket's raw folder (raw-data-zone/) from where it is read and loaded in dataframe. We have taken only 13 columns from the dataset. The null values have been removed from the dataframe , two new columns denoting the period end year and period end month have been created from the period end column, the period end and last updated columns have been dropped, the period end month column has been converted to month name. The final dataframe has been written to transformed folder (transform-data-zone/) in the S3 bucket in parquet format. The ingestion and transformation script has been kept in the S3 bucket's scripts/ folder. <br>

**Provisioning EC2 instance and installing dependencies:** <br>
We have used a t2.medium EC2 instance where we are running our Airflow cluster. Once the EC2 instance is provisioned, we installed python environment and in the environment we installed boto3, awscli. The awscli has been confgured with the access key of the IAM user we are using. Next, we installed packages apache-airflow and apache-airflow-providers-amazon. Once the installation is complete, we started the airflow cluster and logged into the UI using the credentials that airflow created during initialization of the cluster. <br>

**Automation using Airflow:** <br>
Our aim is to orchestrate the emr cluster/termination, data ingestion and transformation for which we are using Airflow. We have created a DAG to configure these steps. As part of the DAG, we have started dummy task of starting pipeline (this is optional but helpful in case of complex DAGs) , we have used EmrCreateJobFlowOperator to create the emr cluster creation task based on the parameters defined in job flow overrides section. In next step , we have used emr job flow sensor to check whether the emr cluster is created or not, the step will check the emr cluster id from previous step and poke until the cluster state changes to Waiting. Once the EMR cluster is provisioned and in waiting state, the next task will start the data extraction and ingestion to S3 bucket. For this task we have used emr step operator which checks the emr cluster id from the cluster creation task and execute the extraction. The location of the ingestion script is passed as argument. It reads the scripts/ folder from the S3 bucket we are using and runs the ingestion script. In next step, we added an emr step sensor which will poke until the extraction step is completed, this task reads cluster id from the cluster creation step and task id of the extraction step. Once the extraction step is completed and the raw data is loaded to the S3 bucket, the transformation step will start. Similar to extraction task, we have used emr step operator for this step as well which reads the cluster id and executes the transformation script. The location of the transformation script is passed as argument. It reads the folder raw-data-zone/ and applies the transformation steps. After this task, we are again using emr step sensor to check the status of the transformation. Once the transformation step is completed, the cluster termination task is started and an emr job flow sensor checks that whether cluster is terminated properly by assesing the cluster state. Once the emr cluster termination is completed, we have set a dummy task to denote end of pipeline. <br>

![Screenshot 2024-07-28 193333](https://github.com/user-attachments/assets/df3c1235-3be4-46ad-85d0-12b20a946266)














