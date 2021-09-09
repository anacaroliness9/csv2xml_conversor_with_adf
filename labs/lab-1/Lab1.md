# Lab 1: Ingest the CSV to the datalake

In this lab you will ingest a csv file in the Azure Data Lake from an on-premises source (your computer) using the Azure Data Factory.

## Microsoft Learn & Technical Documentation

The following Azure services will be used in this lab. If you need further training resources or access to technical documentation please find in the table below links to Microsoft Learn and to each service's Technical Documentation.


Azure Service | Microsoft Learn | Technical Documentation|
--------------|-----------------|------------------------|
Azure Data Lake Storage Gen2 | [Large Scale Data Processing with Azure Data Lake Storage Gen2](https://docs.microsoft.com/en-us/learn/paths/data-processing-with-azure-adls/) | [Azure Data Lake Storage Gen2 Technical Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
Azure Data Factory | [Integrate data with Azure Data Factory or Azure Synapse Pipeline](https://docs.microsoft.com/en-us/learn/modules/data-integration-azure-data-factory/)  | [Azure Data Factory Technical Documentation](https://docs.microsoft.com/en-us/azure/data-factory/)

## Lab Architecture

![](/media/architecture-lab-1.pnG)

Step     | Description
-------- | -----
![1](/media/Black1.png) | Create a Azure Data Lake Storage and its layers
![2](/media/Black2.png) | Create a Azure Data Factory and its ingestion pipeline
![3](/media/Black3.png) | Ingest the CSV file with Azure Data Factory to Azure Data Lake

### Task 1: Create a Azure Data Lake Storage and its layers

> **IMPORTANT:**  Don't forget to create a Resource Group to be used in this entire Lab. For instructions of how to do that, please see this [link](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).

1. First, in your resource group click in +ADD and type *Storage account* to find the resource:
 
 ![1](/media/lab1-1-add-storage.png)

So click in Create button.

2. Provide the details: `Storage account name`, `Region`, `Pricing Tier`, `Redundancy` change to 'Locally-redundant storage (LRS)' and the others leave as is and click on **Next**.
   
3. On **Advanced** below **Security** disable `Enable blob public access` and below **Data Lake Storage Gen2** enable `Enable hierarchical namespace`

4. On **Networking** we can leave the default since we will not deploy this service in a Virtual Network (VNet).
5. On **Data protection** leave as is.
6. Put any Tag if you want.
7. And click on **Create**.

 ![1](/media/lab1-1-create-storage.png)

The deployment process will start and after some minutes we will have the Azure Data Lake Storage Gen 2 created in the Resource Group.

### Create the datalake tiers

Today there is a wide variety of datalake tiers nomenclatures: bronze, silver, gold; raw, trusted, curated  among others. But in the end, it is just different names to represent the same thing. With this [link](https://www.sqlchick.com/entries/2017/12/30/zones-in-a-data-lake) you can know more about them. In our lab, we will use the below nomenclature for the tiers:

 ![1](/media/lab1-3-datalake-tiers.png)

In this datalake we will store de raw csv in the raw tier, then, we will transform this data with the Azure Data Factory and salve it as json in the trusted tier and as last step we will transform this json in a xml file with Azure Functions and store it in the curated tier.

We can create those tiers in several ways, like by the [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/), but if you can't install the Azure Storage Explorer in you computer, you can do through the Azure Portal:

1. Acces the [Azure Portal](https://ms.portal.azure.com/).
   
2. In your resource group acess the storage accoud created previously.
3. On the left pane, in **Data Storage** click on *Containers*.
4. Click on **+ Container**.
5. Fill the field **Name** as *raw* and click on **Create**.
6. Repeate those steps for the *trusted* and the *curated* tiers.

 ![1](/media/lab1-4-datalake-tiers-creation.png)

Now, the tiers of your datalake are created.

### Task 2: Create a Azure Data Factory and its ingestion pipeline