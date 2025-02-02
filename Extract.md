# Extract Layer

## Setting Up the SQL Database

To begin our project, we first need to define the schema for our SQL database, where we will store the dataset extracted from GitHub. I logged into the SQL database using my admin credentials and executed the following query to create a table named `SalesData`:

```sql
CREATE TABLE SalesData (
    Branch_ID VARCHAR(10),
    Dealer_ID VARCHAR(10),
    Model_ID VARCHAR(20),
    Revenue BIGINT,
    Units_Sold BIGINT,
    Date_ID VARCHAR(100),
    Day INT,
    Month INT,
    Year INT,
    BranchName VARCHAR(100),
    DealerName VARCHAR(100)
);
```

![image](https://github.com/user-attachments/assets/38898a0d-4c30-4e38-a732-0519f1f368e8)
  
With this schema in place, we are now ready to proceed with the data ingestion process.

---

## Data Ingestion: Extracting Dataset from GitHub

The next step in our workflow involves extracting the dataset from GitHub and storing it in our SQL database. To achieve this efficiently, we need to establish a connection between GitHub and Azure Data Factory.

### Creating Linked Services

We define two linked services:
1. **GitHub to Azure Data Factory:** This allows us to extract the dataset from GitHub.
2. **Azure Data Factory to SQL Database:** This enables us to load the extracted data into our SQL database.

![image](https://github.com/user-attachments/assets/2eb815fe-2a63-4cda-8bc3-fafea6cf80cf)  

By establishing these linked services, we ensure seamless data transfer from the source (GitHub) to the destination (SQL database) without manual intervention.

---

## Defining the Data Pipeline

Back in Azure Data Factory, I configured the **source** (GitHub) and **sink** (SQL database) for our data pipeline. To make this process dynamic, I implemented a **parameterized dataset**, meaning we don’t have to hardcode the dataset's location. Instead, we simply specify the file name, and the pipeline handles the rest automatically.

![image](https://github.com/user-attachments/assets/7255a0e3-df05-41f7-a347-c76dfbb74065)


Once configured, the dataset is successfully extracted from GitHub and stored in our SQL database. This completes the initial data ingestion phase.

![image](https://github.com/user-attachments/assets/cdfadffe-1cc1-40e1-9286-97a07527972c)
  
---

## Implementing Incremental Loading in Azure Data Factory

  ![image](https://github.com/user-attachments/assets/73ac6ea7-7157-4c5e-9330-9980d5e303ce)
  
One of the most exciting parts of this project is implementing **incremental loading**, which ensures that only new data is loaded into the database, avoiding duplication of older records. To achieve this, we introduce a process known as **update watermark table**.

### Watermark Table

  ![image](https://github.com/user-attachments/assets/e12096ef-aac9-4134-a526-4ac8b2221ba6)

  
We create a new table called `water_table`, which stores the maximum (latest) `Date_ID` from our dataset. Every time new data is loaded, this table is updated to ensure that only new records are inserted.

#### SQL Queries for Watermark Table

```sql
CREATE TABLE water_table(
    last_load VARCHAR(200)
);

SELECT * FROM water_table;

SELECT MAX(Date_ID) FROM [dbo].[SalesData];

INSERT INTO water_table VALUES('DT0000');
```

### Stored Procedure for Updating Watermark Table

```sql
CREATE PROCEDURE updatewatertable
    @lastload VARCHAR(200)
AS
BEGIN
    BEGIN TRANSACTION;

    UPDATE water_table
    SET last_load = @lastload;
    
    COMMIT TRANSACTION;
END;
```

With this setup, the pipeline only processes new records by comparing the latest `Date_ID` in `SalesData` against the `last_load` value in `water_table`. This significantly optimizes data processing and storage.

---

## Data Pipeline Workflow

Here’s how our pipeline operates:
1. **Last Load:** Fetches the last loaded `Date_ID` from `water_table`.
2. **Current Load:** Determines the new records based on the latest `Date_ID`.
3. **Copy Data:** Transfers only new records from GitHub to SQL database.
4. **Stored Procedure:** Updates the `water_table` with the latest `Date_ID`.

  ![image](https://github.com/user-attachments/assets/a2164f91-57ee-48b9-927f-301507241737)
  
This ensures an efficient, automated, and optimized data ingestion process.

---

## Storing Data in Azure Data Lake

As part of our data storage strategy, we save the extracted CSV file in the **bronze container** of the Azure Data Lake in **Parquet format**. Parquet is preferred because of its high efficiency in storage and retrieval.

  ![image](https://github.com/user-attachments/assets/116324b9-404d-41c1-ba44-cca45f01214b)
  
With this final step, our data extraction process is **complete**! 🎉

---

## Conclusion

In this project, I successfully defined a robust SQL database schema, implemented a dynamic data ingestion pipeline from GitHub to SQL using Azure Data Factory, and optimized the process with incremental loading. By leveraging a **watermark table**, we ensure that only new records are processed, significantly improving efficiency.

With all components in place, our data pipeline is now automated, scalable, and ready for future expansions!
[Click here to go to the Transformation Layer](Transformation.md) 
