# Load Layer

In the **Gold layer**, we will create a structured data model and implement it in Databricks. This is where we ensure data is properly formatted, enriched, and ready for analytics.

## **What is a star schema?**  
A star schema is a way of organizing data in a database. It has one main table (called a fact table) in the center, connected to smaller tables (called dimension tables) around it. It looks like a star and makes data analysis faster and easier. |

The star schema for this project:  
![factpng](https://github.com/user-attachments/assets/29c07dee-6d64-4831-991e-d0e61f5d1834)
   

### 📌 Step-by-Step Process:

### 1️⃣ Setting Up the Process
- First, I create an **incremental flag** to determine whether this is an **initial run** (loading all data) or an **incremental run** (only new/updated records).

  ![image](https://github.com/user-attachments/assets/95dcc463-e389-41db-91b1-900ff956debf)
  
- Then, I begin defining our first **dimension model**, which will store model details like `Model_ID`, `Model_Category`, and a **surrogate key** (a unique identifier).

### 2️⃣ Creating the Schema
- If it's the **first time** running the pipeline, I create a **schema** (which just defines column names) and insert an empty structure.

  ![image](https://github.com/user-attachments/assets/f0291d2a-10fc-40af-8738-1bd240eb76b2)
  
- If it's an **incremental run**, I pull actual data from the source instead.

```python
if spark.catalog.tableExists('cars_catalog.gold.dim_model'):
  df_sink = spark.sql("SELECT dim_model_key, Model_ID, model_category FROM parquet.`abfss://silver@shayanathifdatalake.dfs.core.windows.net/carsales`")
else:
  df_sink = spark.sql("SELECT 1 as dim_model_key, Model_ID, model_category FROM parquet.`abfss://silver@shayanathifdatalake.dfs.core.windows.net/carsales` WHERE 1 = 0")
```

  ![image](https://github.com/user-attachments/assets/29388042-0b9f-4520-955b-f5b0d8e97c68)
  
### 3️⃣ Identifying New & Existing Records
- I **compare** the new data with the existing table to separate **new** records from **existing** ones.

```python
df_filter = df_src.join(df_sink, df_src.Model_ID == df_sink.Model_ID, 'left') \
                .select(df_src["Model_ID"], df_src["model_category"], df_sink["dim_model_key"])
```

- **New records** will receive new surrogate keys.
- **Existing records** will simply be updated.

```python
df_old = df_filter.filter(col('dim_model_key').isNotNull())
df_new = df_filter.filter(col('dim_model_key').isNull())
```

### 4️⃣ Assigning Surrogate Keys
- I get the **highest existing surrogate key** to continue numbering from there.

```python
if incremental_flag == '0':
    max_value = 1
else:
    max_val_df = spark.sql("SELECT MAX(dim_model_key) FROM cars_catalog.gold.dim_model")
    max_value = max_val_df.collect()[0][0]
```

- Then, I create the new keys dynamically:

```python
df_filter_new = df_filter_new.withColumn('dim_model_key', max_value + monotonically_increasing_id())
```

  ![image](https://github.com/user-attachments/assets/c85ffe9c-ff36-49cc-aa8d-285950c69237)
  
### 5️⃣ Merging the Data
- I **combine** the updated and new records into a single table:

```python
df_final = df_filter_new.union(df_filter_old)
```

  ![image](https://github.com/user-attachments/assets/7c92297d-74db-488b-9246-bfed42b6f35d)
  
### 6️⃣ Implementing **Slowly Changing Dimension (SCD) Type 1**  
  
## What is Slowly Changing Dimensions Type 1?  
It is a data management technique used in data warehousing to update records when changes occur, without keeping historical data. When a value in a dimension table (such as customer address or product name) changes, the old value is simply replaced with the new one. This method is useful when historical tracking is not required, and only the most recent data is needed.

- **Upsert (Update + Insert) Mechanism:**

```python
if spark.catalog.tableExists('cars_catalog.gold.dim_model'):
    delta_tbl = DeltaTable.forPath(spark, "abfss://gold@shayanathifdatalake.dfs.core.windows.net/dim_model")
    delta_tbl.alias("trg").merge(df_final.alias("src"), "trg.dim_model_key = src.dim_model_key") \
            .whenMatchedUpdateAll() \
            .whenNotMatchedInsertAll() \
            .execute()
else:
    df_final.write.format("delta").mode("overwrite") \
            .option("path", "abfss://gold@shayanathifdatalake.dfs.core.windows.net/dim_model") \
            .saveAsTable("cars_catalog.gold.dim_model")
```

✅ **Now, our `dim_model` table is created and updated as needed!**

### 7️⃣ Repeating the Process for Other Dimensions
- We **repeat the same steps** for other dimension tables (e.g., `dim_branch`, `dim_dealer`, `dim_date`).

  ![image](https://github.com/user-attachments/assets/f67e5c8f-b5c7-4f1f-acad-4d74f9a66ae4)
  
### 8️⃣ Creating the Fact Table
- We **join** the dimension tables to create a comprehensive fact table.

```python
df_fact = df_silver.join(df_dealer, df_silver['Dealer_ID'] == df_dealer['Dealer_ID'], 'left')\
                    .join(df_branch, df_silver['Branch_ID'] == df_branch['Branch_ID'], 'left')\
                    .join(df_model, df_silver['Model_ID'] == df_model['MODEL_ID'], 'left')\
                    .join(df_date, df_silver['Date_ID'] == df_date['Date_ID'], 'left')\
                    .select(df_silver['Revenue'], df_silver['Units_Sold'], df_silver['Revenue_Per_Unit'],
                            df_dealer['Dim_dealer_key'], df_branch['Dim_branch_key'],
                            df_model['Dim_model_key'], df_date['Dim_date_key'])
```

- We then **write the fact table** into Delta Lake.
## What is a delta lake?  
Delta Lake is an open-source storage layer that enhances Apache Spark and big data processing by adding ACID transactions, schema enforcement, and versioning to data lakes. It enables reliable data ingestion, updates, and deletions while ensuring data consistency and high performance. Delta Lake helps organizations manage large-scale data pipelines with better reliability and scalability.


```python
if spark.catalog.tableExists('factsales'):
    deltatbl = DeltaTable.forName('spark','cars_catalog.gold.factsales' )
    deltatbl.alias('trg').merge(df_fact.alias('src'), 
        'trg.dim_branch_key = src.dim_branch_key and 
         trg.dim_dealer_key = src.dim_dealer_key and 
         trg.dim_model_key = src.dim_model_key and 
         trg.dim_date_key = src.dim_date_key') \
        .whenMatchedUpdateAll() \
        .whenNotMatchedInsertAll() \
        .execute()
else:
    df_fact.write.format('delta').mode('Overwrite') \
        .option("path", "abfss://gold@shayanathifdatalake.dfs.core.windows.net/carsales")
```
The fact table:  
![Screenshot 2025-02-01 165138](https://github.com/user-attachments/assets/2679a06c-47dd-4322-98ad-b62fc076b624)
  
🎉 **The Gold Layer is now fully built and ready for analytics!**

---

## 🏁 End-to-End Pipeline Implementation

Now, I will implement an **end-to-end pipeline** using **Azure Data Factory (ADF)** to automate the process.

Let’s revisit the **incremental pipeline** and integrate all the layers together.

  ![image](https://github.com/user-attachments/assets/e5c4b9ec-5ae3-4279-a32e-14a694341bf7)
  
💡 **And just like that, the entire project is complete! 🚀**
