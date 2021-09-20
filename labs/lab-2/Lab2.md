# Lab 2: Transform the data with low code

In this lab you will transform the data with a Wangling Data Flow (Power Query) in Azure Data Factory.

## Microsoft Learn & Technical Documentation

The following Azure services will be used in this lab. If you need further training resources or access to technical documentation please find in the table below links to Microsoft Learn and to each service's Technical Documentation.

Azure Service | Microsoft Learn | Technical Documentation|
--------------|-----------------|------------------------|
Azure Data Factory - Wrangling Data Flow | [Azure Data Factory Data Wrangling](https://www.youtube.com/watch?v=hhfGAzSp-zo)  | [What is data wrangling?](https://docs.microsoft.com/en-us/azure/data-factory/wrangling-overview)

## Lab Architecture

![](/media/architecture-lab-2.png)

Step     | Description
-------- | -----
![1](/media/Black1.png) | Trasforming data with wangling data flow


## Lab Content

<!-- TOC -->

- [Lab 2: Transform the data with low code](#lab-2-transform-the-data-with-low-code)
  - [Microsoft Learn & Technical Documentation](#microsoft-learn--technical-documentation)
  - [Lab Architecture](#lab-architecture)
  - [Lab Content](#lab-content)
    - [Task 1: Transforming data with wangling data flow](#task-1-transforming-data-with-wangling-data-flow)
      - [**1.1. Wrangling data flow creation**](#11-wrangling-data-flow-creation)
      - [**1.2. Transforming the data**](#12-transforming-the-data)

<!-- /TOC -->
<br>

### Task 1: Transforming data with wangling data flow

Now, you will start the development of the trasnformations, and to understand the steps observe **csv** original data format:

```csv
TIME,COMPANY,ID_COMPANY,BENEFIT_CODE,TXT_MONTH_YEAR,QT_MON_BEGIN,QT_MON_ENTER,QT_MON_FOW
20210501,CONTOSO,568,11100,,5375,209,3
20210502,CONTOSO,568,11200,,241,1,2
20210503,CONTOSO,568,11000,,5616,210,5
20210504,CONTOSO,568,12000,,123,10,9
20210505,CONTOSO,568,14000,,473,7,0
```

The result we want in the end, is the following **xml**:

```xml

```

However, as we can't convert the csv to xml with Azure Data Factory, we will the following steps:
- Transform the data using the Power Query, to change the column names, select the only ones you need and create new columns.
- In next lab, we will create the data hierarchy to save in the following json format:
```json

```
- And in the last lab, you will convert the json file to a xml through a Azure Function.

<br>

#### **1.1. Wrangling data flow creation**

1. In your Azure Data Factory, on the left pane, click on <img src="../../media/author-button.png" width="3%" title="Author"/> > **+** > **Power Query**
   
2.  In **Properties** fill the **Name** as *pq_transform_customerdata*.
3.  In **Settings** select *ds_datalakeraw_companydata* on **Dataset**. Now the *Power Query* panel will open:

    ![](/media/lab1-1-powerquery-panel.png)

<br>

#### **1.2. Transforming the data**
<br>

 **Create new columns**

 In the xml file, we have some columns we do not have in the csv, as **year**, **month** and **semester**. To create those columns, follow the steps below:

   - **year**
        1.  Select the **TIME** column, click on **Add column** menu > **Extract** > **First characteres**. 

   ![](/media/lab2-2-pq.png)

   2. In **Number of characters** fill *4*.
   
   ![](/media/lab2-3-pq.png)

    3.  With the right mouse button click on the new column created, and then, click on **Rename** and rename the column as **year** and move to the begining.
   
   ![](/media/lab2-4-pq.png)



    1.  Select the **TIME** column, click on **Add column** menu > **Extract** > **Range**. 

   ![](/media/lab2-5-pq.png)

   2. In **Starting index** fill *4* and in **Number of characteres** fill 2.
   
   ![](/media/lab2-6-pq.png)

    1.  With the right mouse button click on the new column created, and then, click on **Rename** and rename the column as **statistical-balance month**.
    2.  Click in the format column button on the right of the column name **statistical-balance month** and select **Whole number**.
![](/media/lab2-7-pq.png)

    3.  Move the **statistical-balance month** column for the second position
![](/media/lab2-8-pq.png)

        1.  Select the **statistical-balance month** column, click on **Add column** menu > **Clustom column**: 
   
![](/media/lab2-9-pq.png)

 **Alter the columns names**

 **Delete unnecessary columns**


In this step we will alter the following columns names:

Current Name | New name 
--------------|-----------------

1. In your Azure Data Factory, on the left pane, click on <img src="../../media/author-button.png" width="3%" title="Author"/> > **+** > **Power Query**
   
2.  In **Properties** fill the **Name** as *pq_transform_customerdata*.
3.  In **Settings** select *ds_datalakeraw_companydata* on **Dataset**. Now the *Power Query* panel will open:

    ![](/media/lab1-1-powerquery-panel.png)



