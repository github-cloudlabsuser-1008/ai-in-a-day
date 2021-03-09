# Lab 6 - Data monitoring and anomaly detection using Metrics Advisor in Azure Cognitive Services

This lab covers the Metrics Advisor service features from Azure Cognitive Services.

## Task 1 - Explore dashboard of COVID-19 data

Understanding the source datasets is very important in AI and ML. To help you expedite the process, we have created a Power BI dashboard you can use to explore them at the begining of each lab.

![Azure AI in a Day datasets](../media/data-overview-01-01.png)

To get more details about the source datasets, check out the [Data Overview](../data-overview.md) section.

To explore the dashboard of COVID-19 data, open the `Azure-AI-in-a-Day-Data-Overview.pbix` file located on the desktop of the virtual machine provided with your environment.

## Task 2 - Explore lab scenario

Advanced indexing and search work well as long as the corpus of documents contains as little noise as possible. By noise, we identify both issues within documents and whole documents that are not related (or are not close enough, for that matter) to the problem of COVID-19 and its associated domains. In the early stages of document collection, the focus is on the sheer volume (collect as many documents as possible) rather than on quality. However, the system should dismiss documents that are not related to the topics of interest as early as possible.

Using Anomaly Detection and Metrics Advisor, we will demonstrate how to improve the quality of the research document collection process by identifying as early as possible documents that are not related to the problem of COVID-19 and its associated domains.

The following diagram highlights the portion of the general architecture covered by this lab.

![Architecture for Lab 6](./../media/Architecture-6.png)

The high-level steps covered in the lab are:

- Explore dashboard of COVID-19 data
- Explore the lab scenario
- Identify the concept of anomaly detection in a stream of documents
- Preparing the time series data to feed into the Metrics Advisor
- Onboard your time series data in the Metrics Advisor
- Tune the anomaly detection configuration

## Task 3 - Prepare Azure Machine Learning workspace

1. Open the [Azure Portal](https://portal.azure.com) and sign-in with your lab credentials.

2. In the list of your recent resources, locate the Azure Machine Learning workspace, select it, and then select `Launch studio`. If you are prompted to sign-in again, use the same lab credentials you used at the previous step.

![Open Azure Machine Learning Workspace](./media/start-aml-workspace.png)

3. In Azure Machine Learning Studio, select `Compute` from the left side menu and verify that your compute instance is running.

![Verify Azure Machine Learning compute instance is running](./media/check-aml-compute-instance.png)

>Note:
>
>If you launched Azure Machine Learning Studio right after your lab environment was provisioned, you might find the compute instance in a provisioning state. In this case, wait a few minutes until it changes its status to `Running`.

4. From the `Application URI` section associated with the compute instance, select `Jupyter`.

5. In the Jupyter notebook environment, navigate to the folder associated with your lab user.

![Navigate to user folder in Jupyter environment](./media/jupyter-user-folder.png)

6. If the folder does not contain any notebooks, download the following item to your local machine:

[Prepare metrics feed data](https://solliancepublicdata.blob.core.windows.net/ai-in-a-day/lab-06/preparemetricsfeeddata.ipynb)

Upload the file by selecting the `Upload` button from the top right corner of the screen, and then selecting the blue `Upload` button to confirm.

![Upload file to Jupyter notebook environment](./media/upload-file.png)

7. Once the files is uploaded, return to the Azure Portal and select the storage account named `aiinadaystorage...`.

![Locate storage account in Azure Portal](./media/datastore-01.png)

8. Select `Containers` and then select `+ Container` to create a new blob storage container.

![Create new blob storage container](./media/datastore-02.png)

9. Enter `jsonmetrics` as the name, keep all other settings default, and then select `Create` to create the new container.

10. Select `Access keys` from the left side menu, and then select `Show keys`. Save the storage account name and the `key1` value for later use.

![Storage account name and key](./media/datastore-03.png)

## Task 4 - Prepare the COVID cases per age group dataset

1. With the Azure Machine Learning studio and the Jupyter notebook environment open, select the `preparemetricsfeeddata.ipynb` notebook.

   The notebook will guide you through a list of steps needed to prepare a time series-based dataset containing JSON files to be fed into the Metrics Advisor workspace. Each JSON file will contain daily data representing the count of COVID positive cases by age group.

2. Execute the notebook cell by cell (using either Ctrl + Enter to stay on the same cell, or Shift + Enter to advance to the next cell) and observe the results of each cell execution.

## Task 4 - Start your Azure Metrics Advisor environment

1. Open the [Azure Portal](https://portal.azure.com) and sign-in with your lab credentials.

2. In the list of your recent resources, locate the Azure Metrics Advisor workspace and select it. If you are prompted to sign-in again, use the same lab credentials you used at the previous step.
![Open Azure Metrics Advisor](./media/openmetricsadvisor.png)

3. On the Metrics Advisor Quick start page, select the `Go to workspace` link in the first section to start working with the web-based [Metrics Advisor workspace](https://metricsadvisor.azurewebsites.net/).
![Start the web-based workspace](./media/startmetricsadvisor.png)

4. On the Metrics Advisor welcome page, select your Directory, subscription and workspace information and select **Get started**. You are now prepared to create your first Data feed.

## Task 5 - Configure the COVID cases by age group Metrics Advisor data feed 

1. With the Metrics Advidor workspace opened, select the **Add datafeed** option from the left navigation menu.
   
2. Add the data feed by connecting to your time-series data source. Start by selecting the following parameters:
    - **Source type**: `Azure Blob Storage (JSON)`
    - **Granularity**: `Daily`
    - **Ingest data since (UTC)**: `2020-01-01`
    - **Connection string**: provide the connection string from the blob storage access keys page.
  
        ![Get the blob storage connection string](./media/blobstorageconnectionstring.png)

    - **Container**: `jsonmetrics`
    - **Blob template**: `%Y-%m-%d.json` (since the daily json files are provided in with naming format)
    - **JSON format version**: `v2` (since we'll be using the age group dimension in our data schema)

    ![Data feed source properties](./media/adddatafeed.png)

3. Select the **Verify and get schema button** to validate the configured connection.  If there's an error at this step, confirm that your connection string and blob template are correct and your Metrics Advisor instance is able to connect to the data source.
   
4. Once the data schema is loaded and shown like below, configure the appropriate fields as Dimension, Measure or Timestamp.
    ![Schema configuration](./media/schemconfig.png)

5. For **Automatic roll-up** settings, check the `I do not need to include the roll-up analysis for my data` option. 
   
6. Provide the **data feed name**: `covid-ages`. 
   
7. Select **Submit** to confirm and submit the data feed.

   ![Submit schema configuration](./media/submitdatafeed.png)

8. Wait for the ingestion progress dialog and select the **Details** link in order to observe the ingestion log by timestamp.

  ![Check the ingestion progress](./media/ingestionprogress.png)

9. Select the **Visit data-feed: covid-ages** button to navigate to the data feed overview page.
    
10. In the data feed page, select the `count` metric under the **Metrics** section.

    ![Go to the count metric details page](./media/browsemetricdata.png)

11. In the left configuration section, under the **Metric-level configuration** change the default metric-level configuration to mark anomalies when the count value changes to 50% over the previous day count.

![Metric-level configuration](./media/metric-level-configuration.png)