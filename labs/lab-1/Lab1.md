# Lab 1: Ingest the CSV to the datalake

In this lab you will ingest a csv file in the Azure Data Lake from an on-premises source (your computer) using the Azure Data Factory.

## Microsoft Learn & Technical Documentation

The following Azure services will be used in this lab. If you need further training resources or access to technical documentation please find in the table below links to Microsoft Learn and to each service's Technical Documentation.

Azure Service | Microsoft Learn | Technical Documentation|
--------------|-----------------|------------------------|
Azure Data Lake Storage Gen2 | [Large Scale Data Processing with Azure Data Lake Storage Gen2](https://docs.microsoft.com/en-us/learn/paths/data-processing-with-azure-adls/) | [Azure Data Lake Storage Gen2 Technical Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
Azure Data Factory | [Integrate data with Azure Data Factory or Azure Synapse Pipeline](https://docs.microsoft.com/en-us/learn/modules/data-integration-azure-data-factory/)  | [Azure Data Factory Technical Documentation](https://docs.microsoft.com/en-us/azure/data-factory/)

## Lab Architecture

![](/media/architecture-lab-1.png)

Step     | Description
-------- | -----
![1](/media/Black1.png) | Create a Azure Data Lake Storage and its layers
![2](/media/Black2.png) | Create a Azure Data Factory and its ingestion pipeline
![3](/media/Black3.png) | Ingest the CSV file with Azure Data Factory to Azure Data Lake

## Lab Content

<!-- TOC -->

- [Lab 1: Ingest the CSV to the datalake](#lab-1-ingest-the-csv-to-the-datalake)
  - [Microsoft Learn & Technical Documentation](#microsoft-learn--technical-documentation)
  - [Lab Architecture](#lab-architecture)
  - [Lab Content](#lab-content)
    - [Task 1: Create a Azure Data Lake Storage and its layers](#task-1-create-a-azure-data-lake-storage-and-its-layers)
      - [**1.1. Datalake creation**](#11-datalake-creation)
      - [**1.2. Create the datalake tiers**](#12-create-the-datalake-tiers)
    - [Task 2: Create an Azure Data Factory and its ingestion pipeline](#task-2-create-an-azure-data-factory-and-its-ingestion-pipeline)
      - [**2.1. Azure Data Factory Creation**](#21-azure-data-factory-creation)
      - [**2.2. Integration Runtime Self-Hosted Creation**](#22-integration-runtime-self-hosted-creation)
      - [**2.3. On premises Linked Service Creation**](#23-on-premises-linked-service-creation)
      - [**2.4. CSV Dataset On-Premises Creation**](#24-csv-dataset-on-premises-creation)
      - [**2.5. Data Lake Linked Service Creation**](#25-data-lake-linked-service-creation)
      - [**2.6. CSV Dataset Data Lake Raw Creation**](#26-csv-dataset-data-lake-raw-creation)
      - [**2.7. Copy Pipeline**](#27-copy-pipeline)

<!-- /TOC -->
<br>

### Task 1: Create a Azure Data Lake Storage and its layers
<br>

> **IMPORTANT:**  Don't forget to create a Resource Group to be used in this entire Lab. For instructions of how to do that, please see this [link](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).
<br>

#### **1.1. Datalake creation**

1. First, in your resource group click in +ADD and type *Storage account* to find the resource:
 
 ![1](/media/lab1-1-add-storage.png)

So click in Create button.

2. Provide the details: `Storage account name`, `Region`, `Pricing Tier`, `Redundancy` change the `Performance` to *Standard* and the `Redundancy` to *Locally-redundant storage (LRS)* and the others leave as is and click on **Next**.
   
3. On **Advanced** below **Security** disable `Enable blob public access` and below **Data Lake Storage Gen2** enable `Enable hierarchical namespace`

4. On **Networking** we can leave the default since we will not deploy this service in a Virtual Network (VNet).
5. On **Data protection** leave as is.
6. Put any Tag if you want.
7. And click on **Create**.

 ![1](/media/lab1-1-create-storage.png)

The deployment process will start and after some minutes we will have the Azure Data Lake Storage Gen 2 created in the Resource Group.

#### **1.2. Create the datalake tiers**

Today there is a wide variety of datalake tiers nomenclatures: bronze, silver, gold; raw, trusted, curated among others. But in the end, it is just different names to represent the same thing. With this [link](https://www.sqlchick.com/entries/2017/12/30/zones-in-a-data-lake) you can know more about them. In our lab, we will use the below nomenclature for the tiers:

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

### Task 2: Create an Azure Data Factory and its ingestion pipeline

#### **2.1. Azure Data Factory Creation**

1. In your resource group click in +ADD and type *Data Factory* to find the resource:
 
 ![1](/media/lab1-5-add-adf.png)

So click in Create button.

2. Provide the details:  `Region` and `Name` and the others leave as is and click on **Next**.
   
3. On **Git Configuration** enable `Configure Git later`.

1. On **Networking** we can leave the default since we will not deploy this service in a Virtual Network (VNet).
2. On **Advanced** leave as is.
3. Put any Tag if you want.
4. And click on **Create**.

 ![1](/media/lab1-6-create-adf.png)

The deployment process will start and after some minutes we will have the Azure Data Factory created in the Resource Group.

#### **2.2. Integration Runtime Self-Hosted Creation**
<br>
To connect the Data Factory to our on-premises resource the first step is create a integration runtime self-hosted, whitch is the compute infrastructure to the copy activitiy between the on-primeses environment and the cloud data store. To create the IR self-hosted, follow the steps below:
<br>
<br>

> **IMPORTANT:** For you to have a successfully connection beteween your IR and the Azure Data Factory, you need to have some port in you firewall open. You can try the next steps, and if you have connection issues in the next step, you can try to check if you have the permission to open those ports. If you are using a computer with no permission or if you already know you can't open those ports you can upload manually the [data_sample.csv](https://github.com/anacaroliness9/csv2xml_conversor_with_adf/blob/main/data/data_sample.csv) file using the [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) or directly on [Azure Portal](https://ms.portal.azure.com/), accessing you storage resource, clicking on **Containers** > **raw** > **Upload** and uploading this file. If you follow this path you can skip to the [2.5. Data Lake Linked Service Creation](#25-data-lake-linked-service-creation) task.
> 
>  ![1](/media/lab1-upload-csv-manually.png)

<br>

1. When the creation of the Data Factory has finished, go to resource and *Open Azure Data Factory Studio*.

 ![1](/media/lab1-7-open-adf.png)

2. On the left pane, click on the **"Manage" icon** > **Integration runtimes** > **+ New** > **Azure, Self-Hosted** and click on **Continue**.
3. Click on **Self-Hosted**
4. Provide the `Name` and click on **Create**.

 ![1](/media/lab1-8-add-irsh.png)
4. Click on **Click here to launch the express setup for this computer**

 ![1](/media/lab1-9-add-ir-expresssetup.png)
5. Now, the installable integration runtime self-hosted will be download. When has finished, open the file to start the installation.
6. When has finished the installation, copy one of the keys and paste in the IR Self Hosted locally. Then, click on **Register**.
 ![1](/media/lab1-10-add-ir-register.png)
7. Click on **Finish** and wait some minutes to finish the initialization and then click on **Close**.

 ![1](/media/lab1-11-add-ir-register-finish.png)

8. Now you will see the IR Self Hosted with status *Running*.

 ![1](/media/lab1-11-add-ir-running.PNG)

<br>
<br>

#### **2.3. On premises Linked Service Creation**

Now you have to create a linked service, to register the reference for the data origin.

> **IMPORTANT:**  Before start download the files from this Git Repository, you can download directly from Git Hub or you can open you CMD in a folder you choose, and execute this command *"git clone https://github.com/anacaroliness9/csv2xml_conversor_with_adf.git"*. To do that do not forget to [download](https://git-scm.com/downloads) and install the git in your computer.
<br>

1. The fist linked service you will create is for the on-premises origin. In your Azure Data Factory, on the left pane, click on <img src="../../media/manage-button.png" width="3%" title="Manager"/> > **Linked services** > **+New**.
   
2. Search for **File system**, select it and click on **Continue**.
3. Fill the following information:
   
   - Name: ls_filesystem_companydata
   - Connect via integration runtime: IR-SelfHosted
   - Host: Location where your **data_sample.csv** is located. Ex: C:\Users\user1\myfolder\csv2xml_conversor_with_adf\data\
   - Username: The username of you computer
   - Password: The password of you computer

    Then click on **Test connection**, if the connection is ok click on **Save**. If this test fail, you can have the problem presented in the IMPORTANT box in the begining of the [2.2. Integration Runtime Self-Hosted Creation](#22-integration-runtime-self-hosted-creation) task.

<br>

#### **2.4. CSV Dataset On-Premises Creation**

1. In your Azure Data Factory, on the left pane, click on <img src="../../media/author-button.png" width="3%" title="Author"/> > **+** > **Dataset**.
   
2. Search for **File Systen**, select it and click on **Contine**.
3. Select **DelimitedText** format and click on **Continue**.
   - Name: ds_filesystem_companydata
   - Linked service: ls_filesystem_companydata
   - First row as header: Enable
   
   Then, click on **OK**
    
#### **2.5. Data Lake Linked Service Creation**

Now you have to create a linked service, to register the reference for the data origin.

1. The fist linked service you will create is for the on-premises origin. In your Azure Data Factory, on the left pane, click on <img src="../../media/manage-button.png" width="3%" title="Manager"/> > **Linked services** > **+New**.
   
2. Search for **Azure Data Lake Storage Gen2**, select it and click on **Continue**.
3. Fill the following information:
   
   - Name: ls_company_azuredatalake
   - Authentication method: Account Key
   - Account selection method: 
     - From Azure subscription: Your subscription
     - Storage account name: The account name you created in the begining of this lab

    Then click on **Test connection**.
    
     ![1](/media/lab1-13-ir-datalake-linked.PNG)

4. If the connection is ok click on **Save**. 

#### **2.6. CSV Dataset Data Lake Raw Creation**

1. In your Azure Data Factory, on the left pane, click on <img src="../../media/author-button.png" width="3%" title="Author"/>  > **+** > **Dataset**.
   
2. Search for **Azure Data Lake Storage Gen2**, select it and click on **Contine**.
3. Select **DelimitedText** format and click on **Contine**.
   - Name: ds_datalakeraw_companydata
   - Linked service: ls_company_azuredatalake
   - File path:
     - File System: raw
   - First row as header: Enable
   
4. Click on **OK**.
   1. Click on <img src="../../media/publish-button.png" width="20%" title="Publish all"/>.

#### **2.7. Copy Pipeline**
<br>

>IMPORTANT: If you already ingest the file manuaaly to the data lake, you can skip this is step and go to the next [lab](labs/lab-2/Lab2.md).

Now you will create the ingestion pipeline to copy the csv file from the on-premises environment to Azure.

   1. In Azure Data Factory, on the left pane, click on <img src="../../media/author-button.png" width="3%" title="Author"/>  > **+** > **Pipeline**.
   
   2. In **Activities**, expand **Move & Transform** and drag **Copy data** to the pane.

   ![1](/media/lab1-14-copy_data_pipeline.png)

   3. In **Properties** fill the **Name**: *pl_ingestion_companydata*.
   
   4. In **General** tab, fill the **Name**: *copy_companydata*.
   5. In **Source** tab:
      - **Source dataset** select: *ls_filesystem_companydata*
      - **File path type** select: *File filter*
      - **File Filter** fill **csv*  
   6. In **Sink** tab:
      - **Sink dataset** select *ls_company_azuredatalake* 
      - **File extension** as *.csv*.
  
  7. Click on <img src="../../media/publish-button.png" width="20%" title="Publish all"/>.

   8. To run this pipeline, click on **Add trigger** > **Trigger now**.
   
   9.  Now you can go to the [Azure Portal](https://ms.portal.azure.com/) or to [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) to check if you on premises data was copied to the Azure Data Lake Storage successfully.

