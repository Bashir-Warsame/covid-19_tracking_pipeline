
# Data pipeline for tracking C0VID-19 data and dashboarding

Data pipeline for uploading, preprocessing, and visualising COVID-19 data using Google Cloud Platform

## Project Goal

COVID-19 has affected the lives of of everyone for a long  period of time. Therefore it is important to periodiacally track the situation to avoid unexpected outbreaks and to be ready to act. This repository includes implementation of a pipeline for visualization of COVID-19 data: all over the time and the last 14 days. This project builds the pipeline which updates the dashboard for monitoring total cases of COVID19. 

## Content

`src/dbt`: dbt files and folders 

`/images`: printscreens for Readme files

`/infrastructure`: Terraform files

`/scripts`: helper scripts

`/src`: source codes

`env`: file with environmental variables

`/test`: [tests](#Tests) for the code


## Dataset

The worldwide covid data has been provided by [Our World in Data](https://ourworldindata.org/coronavirus).
The source file has been uploaded from [GitHub](https://github.com/owid/covid-19-data) which is daily updated weekly on a Thursday (the source was Johns Hopkins University).

## Dashboarding

![ScreenShot](images/Screenshot%202023-11-24%20at%2015.43.46.png)
![ScreenShot](images/Screenshot%202023-11-23%20at%2019.49.21.png)

## Project Architecture Information

![](images/Screenshot%202023-11-23%20at%2019.10.23.png)

Batch pipeline is implemented using Google Cloud Platform (GCP).
Terraform is used as an IaC (Infrastructure as code) to create resources in GCP, such as virtual machine, Bigquery dataset, google cloud storage bucket and service accounts. The pipeline partially cleans the source csv data, saves it as a parquet file, and moves sequantially first to a datalake, GCP bucket (Google Cloud Storage (GCS)) and then to a data warehouse, Google Biq Query . The whole process is orchestrated by Prefect as a scheduled job every 2 weeks. The data from the data warehouse is then transfromed by dbt for configuring the schema, final cleaning, selecting only the columbs needed, and saving the agregated data as tables in BigQuery. The data is partitioned on the date as the date is then used for quering, this optimizes the process. Due to the size of the data, i close to not cluster it. dbt models used incremental configuration meaning that dbt transforms only the rows in the source data for the last week e.g. rows that have been created or updated since the last time dbt ran.

Dashboard has been built using Looker Studio which is synced with Big Query. Unit tests (/tests)have been written and integrated into CI/CD pipelines via GitHub Actions. The implementation is limited by GCP usage. At the same time, implementation does not involve any local components which makes it more flexible for collaboration goals e.g. working in a team. While local implementation for this particular dataset might be an easier solution (for example, docker + PostgreSQL), cloud implementation provides much more flexibility for team collaboration and production in general.

### Processing the data and putting it into a datalake
The source data is originally in csv file format and is located in Github. It is then sequentually extracted from its source, this process is orchestrated and executed as a scheduled job using Prefect. The jobs are Scheduled in Prefect using deployments. As seen below, the pipeline fetches the dataset, preprocesses it, and saves it as a parquet file to the local storage in this case the local storage will be inside the virtual machine created in GCP. Then, it exectues two steps simultanously running a unit test which checks the schema of a dataframe, and writing the dataset to a GCS bucket.

![ScreenShot](images/Screenshot%202023-11-23%20at%2019.32.14.png)
![ScreenShot](images/Screenshot%202023-11-24%20at%2015.14.03.png)

#### Moving the data from the data lake to a data Warehouse
Once the data is in GCS, it is then moved to the data warehouse the next pipeline does this for us.

![ScreenShot](images/Screenshot%202023-11-23%20at%2019.33.00.png)

##### Transforming the data in the data Warehouse with dbt
Next the data is transformed by dbt for configuring the schema, final cleaning, selecting only thr necessary columns, and saving the resulting data as tables in BigQuery. This is a schema of the architecture in dbt. The dbt model uses incremental configuration which essentially means that dbt transforms only the rows in the source data for the last two weeks. 

![ScreenShot](images/Screenshot%202023-11-23%20at%2018.55.38.png)

#### Reoproduciblity

1. Create a new GCP project account and install the Google SDK on your local machine
2. Use Terraform to setup the GCP infrastructure from your local machine and create a service account
3. clone this repo into your virtual machine and install Anaconda to use virtual environments later.
4. install prefect and set up Blocks in prefect; the Blocks must be created for GCP Bucket and GCP Credentials.
5. Set up your Dbt cloud environment by initializing the project.
6. Run the pipelie manually before deploying it to be schedualed bi weekly delopments.
7. Run in Prefect web_to_gcs and gcs_to_bq fines to have the csv data go from the web to the bq dataset.
8. use dbt run to test created models and check the schema 
9. set up the deployment in dbt cloud by creating a job and have this job run ever 14 days.
10. Use Looker studio to build dashboads of the COVID-19 data










