# Redfin-Housing-Data-pipeline

**Summary:** <br>
In this project, we will be creating an ETL pipeline for analysing Housing market data and use that to draw visualizations. The dataset will be loaded to S3 from where we will run some transformations on it using Pyspark thorugh an EMR cluster, we will load that transformed data to Snowflake using snowpipe and use it draw insisghts and create visualization using Quicksight.


**Dataset Description:** <br>
The project uses housing market dataset from Redfin Data Centre. Redfin is a real estate brokerage, meaning we have direct access to data from local multiple listing services, as well as insight from our real estate agents across USA. The dataset contains listings of various property types across various cities in different states/county in USA, it also lists the inventory details across states and cities and price details of the properties. Redfin offers housing data at National level, metro level, state level, county level, city level, zip codes, neighbourhood level. As part of this project, we are using the dataset from city level.
<br>

**Tech Stack:** <br>
Languages: SQL, Python (PySpark) <br>
Services: AWS S3, Amazon EMR, EC2, VPC, Quicksight, Snowflake, Airflow <br>

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
We have used Pyspark to apply transformation on the data, the raw city level data has been loaded into the S3 bucket's raw folder (raw-data-zone/) from where it is read and loaded in dataframe. We have taken only 13 columns from the dataset. The null values have been removed from the dataframe , two new columns denoting the period end year and period end month have been created from the period end column, the period end and last updated columns have been dropped, the period end month column has been converted to month name. The final dataframe has been written to transformed folder (transform-data-zone/) in the S3 bucket in parquet format. The ingestion and transformation script has been kept in the S3 bucket's scripts/ folder. 












