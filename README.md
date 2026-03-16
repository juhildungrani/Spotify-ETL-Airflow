# Spotify ETL Data Pipeline on AWS

This project is an end-to-end Spotify ETL pipeline built using **Apache Airflow**, **AWS S3**, **AWS Lambda**, and **AWS Glue**. It extracts playlist data from the Spotify API, stores raw JSON files in S3, transforms the data into structured CSV datasets, and orchestrates the workflow using Airflow DAGs. :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1}

## Project Overview

The pipeline performs the following steps:

1. Extracts Spotify playlist track data using the Spotify API.
2. Saves the raw JSON response to Amazon S3.
3. Reads raw files from S3 for processing.
4. Transforms the data into separate datasets for:
   - Albums
   - Artists
   - Songs
5. Stores transformed CSV files back into S3.
6. Moves processed raw files into an archive folder.
7. Uses another Airflow DAG to trigger AWS Lambda and AWS Glue jobs for orchestration and downstream transformation. :contentReference[oaicite:2]{index=2} :contentReference[oaicite:3]{index=3} :contentReference[oaicite:4]{index=4} :contentReference[oaicite:5]{index=5}

---

## Architecture

### DAG 1: `spotify_etl_dag`

This DAG handles the core ETL process:

- Fetch Spotify playlist data
- Upload raw JSON to S3
- Read raw JSON from S3
- Process album, artist, and songs data
- Store transformed CSV files in S3
- Move processed raw files to an archive path in S3 :contentReference[oaicite:6]{index=6}

### DAG 2: `lambda_trigger_dag`

This DAG is used for orchestration and cloud-native processing:

- Triggers an AWS Lambda function for extraction
- Waits for a file to appear in S3 using `S3KeySensor`
- Triggers an AWS Glue job for transformation :contentReference[oaicite:7]{index=7}

---

## Tech Stack

- **Apache Airflow** for workflow orchestration
- **Spotify API** via `spotipy`
- **AWS S3** for raw and transformed data storage
- **AWS Lambda** for serverless extraction
- **AWS Glue** for transformation jobs
- **Python**
- **Pandas** for data processing :contentReference[oaicite:8]{index=8} :contentReference[oaicite:9]{index=9}

---

## Folder / Data Flow Structure

### Raw Data
- `raw_data/to_processed/` — newly extracted Spotify JSON files
- `raw_data/processed/` — archived raw files after processing :contentReference[oaicite:10]{index=10} :contentReference[oaicite:11]{index=11}

### Transformed Data
- `transformed_data/album_data/`
- `transformed_data/artist_data/`
- `transformed_data/songs_data/` :contentReference[oaicite:12]{index=12}

---

## Data Extracted

The pipeline extracts Spotify playlist data from this playlist:

- Spotify Top 50 / trending playlist link stored in the code as:
  `https://open.spotify.com/playlist/37i9dQZEVXbNG2KDcFcKOF?...` :contentReference[oaicite:13]{index=13}

### Output Datasets

#### 1. Album Data
Includes:
- `album_id`
- `name`
- `release_date`
- `total_tracks`
- `url` :contentReference[oaicite:14]{index=14}

#### 2. Artist Data
Includes:
- `artist_id`
- `artist_name`
- `external_url` :contentReference[oaicite:15]{index=15}

#### 3. Songs Data
Includes:
- `song_id`
- `song_name`
- `duration_ms`
- `url`
- `popularity`
- `song_added`
- `album_id`
- `artist_id` :contentReference[oaicite:16]{index=16}

---

## How It Works

### Step 1: Fetch Spotify Data
The pipeline authenticates using Spotify client credentials stored in Airflow variables and fetches playlist track data using the Spotify API. :contentReference[oaicite:17]{index=17}

### Step 2: Store Raw JSON in S3
The raw response is stored in the S3 bucket:

- **Bucket:** `spotify-etl-project-darshil`
- **Path:** `raw_data/to_processed/` :contentReference[oaicite:18]{index=18}

### Step 3: Read Raw Data from S3
The pipeline reads all JSON files from the raw folder for further processing. :contentReference[oaicite:19]{index=19}

### Step 4: Transform Data
The raw data is split into album, artist, and songs datasets using Pandas. Duplicate album and artist records are removed, and date columns are converted into datetime format. :contentReference[oaicite:20]{index=20} :contentReference[oaicite:21]{index=21} :contentReference[oaicite:22]{index=22}

### Step 5: Save Transformed Data
Each transformed dataset is uploaded to S3 as CSV files under separate folders. :contentReference[oaicite:23]{index=23}

### Step 6: Archive Processed Raw Files
After successful transformation, raw files are moved from `raw_data/to_processed/` to `raw_data/processed/`. :contentReference[oaicite:24]{index=24}

### Step 7: Trigger Lambda and Glue
The second DAG can trigger Lambda functions and an AWS Glue job to automate extraction and transformation workflows. It also uses an S3 sensor to ensure files are uploaded before continuing. :contentReference[oaicite:25]{index=25}

---

## Prerequisites

Before running this project, make sure you have:

- An Airflow environment configured
- AWS credentials configured in Airflow connection `aws_s3_airbnb`
- Spotify API credentials stored in Airflow variables:
  - `spotify_client_id`
  - `spotify_client_secret`
- Access to:
  - AWS S3
  - AWS Lambda
  - AWS Glue :contentReference[oaicite:26]{index=26} :contentReference[oaicite:27]{index=27}

---

## Airflow Connections and Variables

### Required Airflow Connection
- `aws_s3_airbnb` — used for S3, Lambda, and Glue integration. :contentReference[oaicite:28]{index=28} :contentReference[oaicite:29]{index=29}

### Required Airflow Variables
- `spotify_client_id`
- `spotify_client_secret` :contentReference[oaicite:30]{index=30}

---

## AWS Resources Used

### S3
- Bucket: `spotify-etl-project-darshil` :contentReference[oaicite:31]{index=31}

### Lambda
- Function: `spotify_data_extract` :contentReference[oaicite:32]{index=32}

### Glue
- Job: `spotify_transformation_job`
- IAM Role: `spotify_glue_iam_role`
- Script Location:  
  `s3://aws-glue-assets-206986907456-ap-south-1/scripts/spotify_transformation_job.py` :contentReference[oaicite:33]{index=33}

---

## DAG Schedule

Both DAGs are configured to run **daily** with `catchup=False`. :contentReference[oaicite:34]{index=34} :contentReference[oaicite:35]{index=35}

---

## Project Files

- `spotify_data_pipeline.py` — main ETL DAG that fetches, processes, and stores Spotify data. :contentReference[oaicite:36]{index=36}
- `spotify_lambda.py` — orchestration DAG that triggers Lambda, checks S3, and starts Glue jobs. :contentReference[oaicite:37]{index=37}

---

## Possible Improvements

- Add logging and monitoring
- Add retry and failure alerting
- Parameterize playlist ID
- Add unit tests for transformations
- Store output in Parquet instead of CSV for analytics efficiency
- Integrate with a data warehouse like Snowflake or Redshift

---

## Author

Built as a cloud-based Spotify ETL pipeline project using Airflow and AWS services.
