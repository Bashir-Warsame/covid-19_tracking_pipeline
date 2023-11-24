
# Data pipeline for tracking C0VID-19 data and dashboarding

Data pipeline for uploading, preprocessing, and visualising COVID-19 data using Google Cloud Platform
This repository includes implementation of a pipeline for visualization of COVID-19 data: all over the time and the last 14 days.
Original idea of this pipeline is to have scheduled jobs with regularly updated table every two weeks. COVID19 has been affected our lives for quite long period of time already. It is important to regularly track the situation to avoid unexpected situations and be ready for actions beforehand.This project builds the pipeline which updates the dashboard for monitoring total cases of COVID19. 

## Dataset

The worldwide covid data has been provided by [Our World in Data](https://ourworldindata.org/coronavirus).
The source file has been uploaded from [GitHub](https://github.com/owid/covid-19-data) which is daily updated weekly on a Thursday (the source was Johns Hopkins University).

## Project Architecture 

![](images/Screenshot%202023-11-23%20at%2019.10.23.png)

The source data (raw level) is originally in csv format and located in GitHub.
Batch pipeline is implemented using Google Cloud Platform (GCP).
Terraform is used as an IaC (Infrastructure as code) to create resources in GCP, such as virtual machine, Bigquery dataset, google cloud storage bucket, service accounts etc

Pipeline partially cleans the source csv data, saves it as a parquet file, and moves sequantially first to a datalake, GCP bucket (Google Cloud Storage (GCS)) and then to a data warehouse, Google Biq Query . The whole process is orchestrated by Prefect as a scheduled job every 2 weeks.

dbt models used incremental configuration meaning that dbt transforms only the rows in the source data for the last week e.g. rows that have been created or updated since the last time dbt ran.

Dashboard has been built using Looker Studio which is synced with Big Query.

Unit tests (/tests)have been written and integrated into CI/CD pipelines via GitHub Actions.

The implementation is limited by GCP usage. At the same time, implementation does not involve any local components which makes it more flexible for collaboration goals e.g. working in a team.

While local implementation for this particular dataset might be an easier solution (for example, docker + PostgreSQL), cloud implementation provides much more flexibility for team collaboration and production in general.

# Processing the dataset sending it into a datalake

Data source abd data lake from the diagram indicate this part. The source data (raw data from github) is originally in csv format and located in GitHub.







## Dashboard

![ScreenShot](images/Screenshot%202023-11-24%20at%2015.43.46.png)
![ScreenShot](images/Screenshot%202023-11-23%20at%2019.49.21.png)
