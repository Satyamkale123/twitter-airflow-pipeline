# Twitter Data Pipeline with Apache Airflow (AWS EC2 → S3)

Automated pipeline that extracts tweets via **Tweepy**, transforms to a clean tabular format with **Pandas**, and orchestrates the workflow using **Apache Airflow** on **AWS EC2**, landing daily CSVs in **Amazon S3**.

![Architecture](T_A_architecture.png)

---

## Why this project matters

- **End-to-end ownership**: I designed, coded, deployed, scheduled, and monitored a cloud data pipeline.
- **Real orchestration**: Shows understanding of DAGs, retries, scheduling, and task observability in Airflow.
- **Cloud-native storage**: Production-style landing in S3 (partitioning by date is easy to add).
- **Security & DevEx**: Uses IAM roles for EC2→S3 and environment variables for keys (no secrets in code).

---

## Tech Stack

- **Python 3.x**: `pandas`, `tweepy`, `s3fs`
- **Apache Airflow** (2.x) on **AWS EC2**
- **Amazon S3** for data lake storage
- **IAM Role** for secure, keyless S3 access from EC2
- (Optional) **Docker/virtualenv** for reproducible environments

---

## 🚀 End-to-End Steps (From Data Extraction to Final Storage)

**Data Flow:** 🐦 Twitter API → 🐍 Python ETL (Tweepy + Pandas) → ⚙️ Apache Airflow (on AWS EC2) → ☁️ Amazon S3

---

### **1️⃣ Define the Data to Pull (Twitter)**
- Identified the target Twitter account (e.g., `@elonmusk`) and key fields such as:
  - Tweet text, likes, retweets, and creation timestamps.
- **Outcome:** Clear data extraction plan and API request design.

---

### **2️⃣ Obtain Twitter API Access**
- Created a Twitter Developer App and generated the required:
  - Consumer Key / Secret
  - Access Token / Secret
- **Outcome:** Authorized credentials to connect securely to the Twitter API.

---

### **3️⃣ Build the ETL Logic (Python)**
- Designed a Python function to:
  1. Authenticate to Twitter API.
  2. Fetch the latest N tweets (excluding retweets/replies).
  3. Normalize and clean data using Pandas (select columns, format dates, rename fields).
- **Outcome:** Reusable ETL logic returning a structured dataset.

---

### **4️⃣ Set Up the Storage Target (Amazon S3)**
- Created an **S3 bucket** for storing daily tweet data.
- Designed a partitioned folder structure:  
  `s3://my-twitter-bucket/twitter/dt=YYYY-MM-DD/refined_tweets.csv`
- **Outcome:** Defined cloud storage location for daily outputs.

---

### **5️⃣ Launch the Orchestration Host (AWS EC2)**
- Provisioned an **EC2 instance** to host Apache Airflow.
- Configured security groups to allow:
  - Port **22 (SSH)** for secure connection.
  - Port **8080 (Airflow UI)** for web access.
- **Outcome:** Compute environment for workflow orchestration.

---

### **6️⃣ Grant Secure S3 Access (IAM Role)**
- Created an **IAM Role** and attached it to the EC2 instance with:
  - `s3:ListBucket` and `s3:PutObject` permissions.
- **Connection Detail:**  
  EC2 communicates with S3 using this role — no static AWS credentials needed.
- **Outcome:** Secure, keyless S3 access for EC2 and Airflow.

---

### **7️⃣ Install Apache Airflow and Dependencies**
- Installed Airflow and required Python packages (`pandas`, `tweepy`, `s3fs`) on the EC2 instance.
- Initialized Airflow (`airflow db init`) and started both **webserver** and **scheduler**.
- **Outcome:** Airflow environment ready to orchestrate ETL tasks.

---

### **8️⃣ Deploy DAG and ETL Script into Airflow**
- Copied both scripts into the Airflow DAGs directory:
  - `twitter_dag.py` → defines the daily schedule and task.
  - `twitter_etl.py` → performs the extract-transform-load operations.
- **Connection Detail:**  
  Airflow discovers these files, registers the DAG, and executes the ETL script automatically.
- **Outcome:** Pipeline becomes visible in the Airflow UI.

---

### **9️⃣ Configure Environment Variables and Secrets**
- Stored API keys and configuration values (handle, tweet limit, S3 bucket) using:
  - **Environment variables** on EC2, or  
  - **Airflow Variables / Connections** within the UI.
- **Connection Detail:**  
  - ETL ↔ Twitter API → uses Developer Credentials  
  - ETL ↔ S3 → uses IAM Role from EC2
- **Outcome:** Secure runtime configuration without hardcoding credentials.

---

### **🔟 Run and Validate the Pipeline**
- Triggered the **DAG manually** in Airflow for the first test run.
- Checked task logs to ensure:
  - Successful Twitter data extraction
  - Proper transformation
  - Data successfully written to S3
- **Outcome:** Verified that end-to-end data flow works as expected.

---

### **11️⃣ Schedule Automated Daily Runs**
- Enabled the DAG to run **once per day** automatically.
- **Connection Detail:**  
  Airflow Scheduler triggers the ETL daily → ETL writes a new timestamped file to S3.
- **Outcome:** Fully automated daily data ingestion.

---

### **12️⃣ Monitor and Troubleshoot**
- Used Airflow’s **Graph View**, **Tree View**, and **Logs** for monitoring.
- Configured **retry logic** for fault tolerance.
- (Optional) Email/Slack alerts can be added for failed runs.
- **Outcome:** Achieved observability and reliability in production.

---

### **13️⃣ Validate and Improve Data Quality**
- Validated output files in S3 for:
  - Correct column schema
  - Expected row counts
  - No missing timestamps
- Planned future upgrades:
  - Convert CSV → Parquet for better performance
  - Add schema validation (using `pandera` or `great_expectations`)
- **Outcome:** Reliable and analysis-ready datasets.

---

### **14️⃣ Cost Optimization & Security Maintenance**
- Stopped/terminated EC2 instance when not in use.
- Applied S3 **lifecycle policies** for archival or auto-deletion of old data.
- Restricted **Security Group** access to personal IP only.
- **Outcome:** Lower cost, safer deployment, and minimal attack surface.

---

## 🔗 How Services Are Connected

| Integration | Description |
|--------------|-------------|
| **Python ETL ↔ Twitter API** | Tweepy authenticates via Twitter Developer credentials and fetches tweets as JSON data. |
| **Airflow ↔ Python ETL** | Airflow executes the Python ETL function as part of a daily DAG. |
| **EC2 ↔ S3** | The EC2 instance uploads files to S3 using its attached IAM Role. |
| **User ↔ Airflow UI** | The developer monitors DAGs, runs, and logs through the Airflow web interface on port 8080. |

---

## ✅ Final Outcome

- A **daily automated data pipeline** that:
  - Extracts live tweets from the Twitter API  
  - Cleans and structures data using Pandas  
  - Schedules and monitors runs with Apache Airflow  
  - Stores clean, partitioned CSVs in Amazon S3 for analytics

**Result:** A production-style, end-to-end cloud data pipeline demonstrating Data Engineering skills in ETL, orchestration, and AWS integration.


