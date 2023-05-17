# Formula1-Big-Data-Project-With-Azure-Databricks

This Project is about building a Cloud data platform for the reporting and analysis of Formula1 Motor Sport data using Azure Databricks

Azure Databricks is a unified set of tools for building, deploying, sharing and maintaining enterprise-grade data solutions at scale.

The Databricks Lakehouse Platform integrates with various cloud storages coupled with the management and deployment of cloud infrastructures on your behalf.

### Azure Resources Used for this Project
* Azure Data Lake Storage
* Azure Data Factory
* Azure Databricks
* Azure Key Vault

### Project Requirements
The requirements for this project is broken down into six different parts which are;

#### 1. Data Ingestion Requirements
* Ingest aall 8 files into Azure data lake. 
* Ingested data must have the same schema applied.
* Ingested data must have audit columns.
* Ingested data must be stored in  columnar format (i.e parquet).
* We must be able to analyze the ingested data via SQL.
* Ingestion Logic must be able to handle incremental load.

#### 2. Data Transformation Requirements
* Join the key information required for reporting to create a new table.
* Join the key information required for analysis to create a new table.
* Transformed tables must have audit columns.
* We must be able to analyze the transformed data via SQL.
* Transformed data must be stored in columnar format (i.e parquet).
* Transformation logic must be able to handle incremental load.

#### 3. Data Reporting Requirements
* We want to be able to know Driver Standings.
* We should be able to know Constructor Standings as well.

#### 4. Data Analysis Requirements
* We want to know the Dominant drivers.
* Dominant Teams. 
* Visualize the Outputs.
* Create Databricks dashboards.

#### 5. Scheduling Requirements
* Scheduled to run every sunday at 10pm.
* Ability to monitor pipelines.
* Ability to rerun failed pipelines.
* Ability to set up alerts on failures

#### 6. Other Non-Functional Requirements
* Ability to delete individual records
* Ability to see history and time travel
* Ability to roll back to a previous version

### Solution Architecture of this Project
The first thing was to ingest the Data from the Ergast API using Azure Data Factory into ADLS Raw layer. This was followed by another ingestion and transformation from the Raw layer to the Processed layer using Databricks.

The data in this layer will have the schema applied as well as being stored in columnar formats i.e parquet. Partitions will also be created where applicable as well.

This data will also be converted from parquet to delta lake to meet some of the non-functional requirements.

The data in the processed layer is then transformed further to meet the business requirements into the Presentation layer. It will also be in parquet and delta lake format as well.

We then use databricks notebooks to analyze the data and create dashboards. We also connected Power BI for more sophisticated dashboard generation before finally scheduling the pipeline with Azure Data Factory Pipelines.
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/solution%20architecture.png">

### Solution
The first thing we did was to create three containers in our ADLS named *raw*, *processed* and *presentation*. These containers was then mounted to databricks using Azure Service principal and Databricks Secret scopes as shown below.
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/mount%20storages.png)

After mounting, we then uploaded the raw datasets manually into to the raw container, we could have use ADF for the ingestion but we just decided to do it manually for simplicity sake.

We then used our databricks notebook to create a f1_raw database so that we can create external tables on top of the raw datasets inside the f1_raw database as seen below.
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/Create%20raw%20database%20and%20tables.png)
Note that we created external tables for all the raw datasets so that we can ustilize the *DeltaLakeHouse Architecture* of Databricks.

We also created f1_processed and f1_presentation databases which will house our managed tables from the files that will be transformed later,
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/create%20processed%20%26%20presentation%20databases.png)

Each file was then ingested and transformed from the raw layer to the processed layer. The files are of different file formats which are csv, json and some are in folders as well.
This implies that they are ingested and transformed based on their different file formats as shown below.

First we ingest all the datasets based on their file formats and requirements. 
We ingest the csv file followed by the json file respectively.
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/ingest%20csv.png">

![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/ingest%20json.png)

After injesting all the 8 files, we then transformed the datasets as required
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/tranform1.png)

![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/transform2.png)

Transofrmation of all the datasets follows the same approach as shown above.

It is worthy to note that we have 8 files to be ingested and transformed, 4 of those files are implemented using full loads while the other four are implemented using incremental loads.
The way we handle full or bulk loads are different from how we handle incremental loads.

After transformation, we then write the data with full loads to the processed containers in parquet format using the *saveAsTable* syntax which also saves the data as a table in our f1_processed database.
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/full%20load.png)

For the incremental load, we have to approach it by utilizing the advantages of Databricks lakehouse architecture which allows us to append data by using the *merge* syntax incrementally.
We first define the merge statement as a function as shown below;
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/merge.png">

Then we load all the four incremental load data by calling the function as shown below
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/incremental%20load.png">

Now we have all our data in the processed layer and as well as managed tables in our f1_processed database as well. The next thing is to transform the data further to meet BI reporting inside the presentation layer while also creating tables in our f1_presentation database.
We created the race_results, drivers and constructors standings which will allow us perform reporting to meet the reporting requirements as shown below.

To create the presentation data for our BI reporting, we read the various data needed for the transformation as shown;
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/prentation%20read.png">

Next we join them and selected the required columns as seen below;
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/presentation%20join.png">

After that, we then write the data to the presentation layer using the incremental load approach we used which will also create managed tables in the f1_presentation database that was created earlier.
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/presentation%20load.png">
The other presentation files followed the same approach as above.

We then analyzed the dataset to find out the dominant drivers and dominant teams over the years.
###### Dominant Drivers Analysis and Visualization
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/drivers%20analysis.png)
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/dominant_drivers_viz.png)

##### Dominant Teams Analysis and Visualization
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/team%20analysis.png)
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/dominant_teams_viz.png)

For a more sophisticated reporting, we could connect our databricks workspace to Power BI as well as shown below;
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/connect_to_powerbi_1.png)
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/connect_to_powerbi_2.png)
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/connect_to_powerbi_3.png)

To meet up with our scheduling and orchestration requirements, we have to use ADF pipelines.
We created pipelines for the ingestion files and transformatipn files as shown below;
![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/pipeline_ingestion_all_files.png)

![](https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/pipeline_tranform_files.png)

We then tied both pipelines in a master pipeline with trigger to automate the whole process as shown below;
<img src="https://github.com/jaykay04/Formula1_Big_Data_Project_Using_Azure_Databricks/blob/main/Images/trigger_master_pipeline.png">

In conclusion, we are able utilize the power of Azure Databricks LakeHouse Architecture to Extract data of different file formats, Transform them and Load them based on full and incrememtal loads.
