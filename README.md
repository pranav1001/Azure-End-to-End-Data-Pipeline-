# Azure End-to-End Data Pipeline for Tokyo Olympics Analytics

This project demonstrates the construction of a complete end-to-end data pipeline using Microsoft Azure services. The pipeline ingests raw data about the Tokyo 2020 Olympics from a remote source, processes and transforms it, stores it in a data lake, enables analytical querying, and finally visualizes the insights through an interactive dashboard.

![Final Dashboard](https://i.imgur.com/8a5X2Yf.png)
*Final Power BI Dashboard*

---

## 🏛️ Project Architecture

The pipeline follows a modern data warehousing architecture, leveraging several powerful Azure services in sequence:

```mermaid
graph TD
    A[<B>Raw Data</B><br>(CSV Files on GitHub)] --> B{<B>Azure Data Factory</B><br>(Orchestration)};
    B --> C[<B>Azure Data Lake Storage Gen2</B><br>(Raw Data Container)];
    C --> D{<B>Azure Databricks</B><br>(Transformation with PySpark)};
    D --> E[<B>Azure Data Lake Storage Gen2</B><br>(Transformed Data Container)];
    E --> F{<B>Azure Synapse Analytics</B><br>(Data Warehousing & SQL Queries)};
    F --> G[<B>Power BI</B><br>(Visualization & Dashboarding)];
```

---

## 📊 Dataset

The project uses the **Tokyo 2020 Olympics Dataset**, which includes detailed information about:
- **Athletes:** Over 11,000 participating athletes.
- **Coaches:** Details of the coaching staff.
- **Teams:** 743 teams across various sports.
- **Medals:** Gold, Silver, and Bronze medal winners.
- **Gender Entries:** Breakdown of participation by gender.
- **Disciplines:** 47 different sporting disciplines.

The raw data is stored in multiple CSV files, which are ingested directly from a GitHub repository.

---

## 🛠️ Technologies Used

- **Orchestration:** [Azure Data Factory (ADF)](https://azure.microsoft.com/en-us/products/data-factory/)
- **Data Storage:** [Azure Data Lake Storage Gen2 (ADLS)](https://azure.microsoft.com/en-us/products/storage/data-lake-storage/)
- **Data Transformation:** [Azure Databricks](https://azure.microsoft.com/en-us/products/databricks/) (using PySpark)
- **Data Warehousing:** [Azure Synapse Analytics](https://azure.microsoft.com/en-us/products/synapse-analytics/)
- **Data Visualization:** [Microsoft Power BI](https://powerbi.microsoft.com/en-us/)

---

## ⚙️ Project Workflow

The pipeline is executed in four main stages:

### 1. Data Ingestion
- An **Azure Data Factory pipeline** is triggered to begin the process.
- The `Copy data` activity in ADF connects to the raw CSV files stored publicly on GitHub via an HTTP linked service.
- The data is ingested and loaded into a `raw-data` container within **Azure Data Lake Storage Gen2** without modification. This process is repeated for all five source files (Athletes, Coaches, Medals, Teams, EntriesGender).

### 2. Data Transformation
- An **Azure Databricks** workspace is used to clean and transform the raw data.
- The ADLS Gen2 storage is mounted onto the Databricks workspace using an **App Registration** (Service Principal) and OAuth 2.0 for secure access.
- A PySpark notebook is executed to perform transformations, including:
  - Correcting data types.
  - Handling missing values and duplicates.
  - Ensuring schema consistency.
- The cleaned, transformed data is then written back to the ADLS Gen2 in the `transformed-data` container, partitioned and ready for analysis.

### 3. Data Warehousing and Serving
- An **Azure Synapse Analytics** workspace is connected to the `transformed-data` in the data lake.
- A new Lake database is created within Synapse.
- External tables are created on top of the transformed CSV files, allowing the data to be queried directly using standard SQL. This provides a high-performance analytical engine without moving the data.

### 4. Data Visualization
- **Power BI Desktop** is connected to the Azure Synapse Analytics workspace.
- SQL queries are run from Power BI against the Synapse external tables to import the data.
- An interactive dashboard is built to explore the data, featuring key insights such as:
  - Medal counts by country.
  - Number of athletes per country and discipline.
  - Gender distribution across the games.
  - Overall participation statistics.

---

## 🚀 How to Replicate This Project

To set up and run this pipeline yourself, follow these steps:

### Prerequisites
- An active Microsoft Azure subscription.
- Power BI Desktop installed.
- The raw dataset files available in a public GitHub repository.

### Setup Instructions

1.  **Create Storage Account:**
    - In the Azure Portal, create a new **Storage Account**.
    - In the `Advanced` tab during creation, **enable the Hierarchical Namespace** to make it an Azure Data Lake Storage Gen2 account.
    - Create two containers inside the storage account: `raw-data` and `transformed-data`.

2.  **Set Up Data Factory:**
    - Create a new **Azure Data Factory** resource.
    - Launch the ADF Studio and create a new pipeline.
    - For each of the five CSV files, add a `Copy data` activity:
      - **Source:** Create a new dataset using the `HTTP` linked service, pointing to the raw URL of the CSV file on GitHub.
      - **Sink:** Create a new dataset pointing to the `raw-data` container in your ADLS Gen2 account.
    - Chain the activities so they run sequentially. Publish and trigger the pipeline to ingest the data.

3.  **Configure Databricks Workspace:**
    - **App Registration:** Create a new App Registration in Azure Active Directory to grant Databricks programmatic access to the storage account.
    - **Create a Client Secret** for the app and securely copy its value.
    - **Assign Role:** In your ADLS Gen2 account, go to `Access Control (IAM)` and assign the `Storage Blob Data Contributor` role to the App Registration you created.
    - **Create Databricks Workspace:** Deploy a new Azure Databricks workspace.
    - **Create a Cluster:** Inside the workspace, create a standard compute cluster.
    - **Mount Storage:** Create a new notebook and use the provided Python code to mount your ADLS Gen2 container to the Databricks File System (`/mnt/`). You will need the Application (client) ID, Directory (tenant) ID, and the client secret from your App Registration.
    - **Run Transformation:** Add the PySpark code to the notebook to read from the `raw-data` mount, perform transformations, and write the output to the `transformed-data` mount.

4.  **Set Up Synapse Analytics:**
    - Create a new **Azure Synapse Analytics** workspace.
    - In the Synapse Studio, navigate to the `Data` tab.
    - Connect to your ADLS Gen2 account as a linked service.
    - Create a new **Lake Database**.
    - Right-click the transformed CSV files in your `transformed-data` container and select the option to create new **external tables**.

5.  **Build the Dashboard:**
    - Open Power BI Desktop.
    - Select `Get Data` and connect to your **Azure Synapse Analytics** workspace.
    - Write SQL queries to import the data from the external tables you created.
    - Build your visualizations and reports.

---

## 📄 License

This project is open-source and available for anyone to use and modify.
