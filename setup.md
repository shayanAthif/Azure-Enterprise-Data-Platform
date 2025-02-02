# My Azure Setup Walkthrough  

I began by creating a **resource group** – a logical container to organize Azure services like databases and storage for my project. Named it `shayanathif-data-project`. While the name only needs to be unique within *my* Azure subscription (not globally), I still picked something descriptive. Selected **South India** as the region since it’s closest to me, which reduces latency.    
<br>
![image](https://github.com/user-attachments/assets/7c3b04b5-2239-4ef3-b765-34e981ed8f24)


---

## **Creating the Data Lake (Storage Account)**  
Next, I built a **data lake** by creating a storage account:  
1. Searched for *Storage Account* in Azure and clicked *Create*.  
2. Selected my existing resource group.  
3. Gave the storage account name as `shayanathifdatalake` (Must be unique;Azure enforces this).  
4. Kept the region as **South India** for consistency.  
5. Chose **Standard** performance (cost-effective) and **Locally Redundant Storage (LRS)** – stores 3 copies of data in one datacenter.


![image](https://github.com/user-attachments/assets/7d5de955-0195-4d3f-8529-4d3498c7210a)
<br>  
*‼️‼️‼️ VERY VERY IMPORTANT ‼️‼️‼️*  
Under the *Advanced* tab, I enabled **hierarchical namespace**. This is **critical** – without it, Azure creates a basic *Blob Storage* account instead of a data lake. Blob Storage is simpler, designed for unstructured files (images, documents), but lacks folder structures needed for analytics. Enabling hierarchical namespace converts it to **Azure Data Lake Gen2**, which organizes files into directories (like a traditional filesystem). Clicked *Review + Create* and finalized it.  

---

## **Setting Up Data Factory**  
For automating data workflows, I created **Azure Data Factory**:  
1. Searched for *Data Factory* in Azure.  
2. Selected my resource group, name (`shayanathif-datafactory`) and region (**South India**).  
3. Skipped advanced settings (kept defaults) and clicked *Create*.  
This will later help orchestrate pipelines to move data between services.
  
![image](https://github.com/user-attachments/assets/36122e0c-6868-40c5-95ac-eaa3cd5d31ef)


---

## **Configuring the SQL Database**  
This part was trickier. Here’s what I did:  
1. Navigated to **Azure SQL** and chose *SQL Database* (single database). Skipped *Elastic Pool* (used to share resources across multiple databases – unnecessary for my single-database project) and *SQL VM* (self-managed virtual machines).  
2. Selected my resource group and named the database.  
3. **Region Issues**: Surprisingly, most regions were unavailable. Ended up selecting **Germany West Central** (Azure sometimes restricts regions based on resource availability).  
4. Created a new **server** (logical container for databases):  
   - Gave it a unique server name.  
   - Set location to **Germany West Central** (to match the database).  
   - Authentication: Chose **SQL + Microsoft Entra** (hybrid for flexibility).  
   - Assigned myself as the Entra admin and set a SQL admin username/password.  

Under database configuration:  
- **Workload Environment**: Selected *Development* (lower cost, limited performance – ideal for testing).  
- **Backup Redundancy**: Chose *Locally Redundant (LRS)*. Other options:  
    
![image](https://github.com/user-attachments/assets/7024c274-2c0e-4bed-9c5b-87e8009bc703)


In *Networking*, enabled **Public Endpoint** for easy access (⚠️ risky for production but okay for short-term testing).  

**Cost Saver**: Opted for the **free tier** offering:  
- 100,000 vCore seconds/month  
- 32 GB database storage  
- 32 GB backup storage  
Without this, it would’ve cost ~474 INR/month.  
  
![image](https://github.com/user-attachments/assets/286a7bff-6ab8-41fa-9b98-e5bb05aee525)

Clicked *Review + Create*, finalized settings, and deployed.  

---

## **Final Setup**  
Now that our setup is complete, let's review the resource group.  
![image](https://github.com/user-attachments/assets/1ca69a28-91c9-4de1-9173-bd6ffe550547)


Here, we can see the data lake, the data factory, databaseand the database server.

Now, let's move on to the next step.
[Click here to go to the Extract Layer](Extract.md) 
