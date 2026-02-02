# Google Cloud Storage Backup & Restore Guide

This document describes **step-by-step instructions** to back up and restore data between Google Cloud Storage (GCS) buckets using **Google Cloud SDK (`gcloud`)**.

---

## 1. VM Instance Setup

Create a Virtual Machine (VM) to perform backup and restore operations.

### VM Configuration
- **Region:** `asia-southeast1` (Singapore)
- **OS:** Ubuntu 22.04 LTS
- **Disk Size:** 30 GB
- **Networking:**
  - ✅ Allow HTTP traffic
  - ✅ Allow HTTPS traffic

---

## 2. Prerequisites

Make sure the following requirements are met:

- Google Cloud SDK installed  
  ```bash
  sudo apt update
  sudo apt install -y google-cloud-cli
  ```

- Verify installation:
  ```bash
  gcloud version
  ```

- Access to:
  - Old GCP account (source bucket)
  - New GCP account (destination bucket)

---

## 3. Backup Google Cloud Storage (Old GCP Account)

### 3.1 Login to Old GCP Account
Authenticate with the old GCP account that owns the source bucket:

```bash
gcloud auth login
```

(Optional) Set correct project:
```bash
gcloud config set project OLD_PROJECT_ID
```

---

### 3.2 Verify Source Bucket
Check that the bucket exists and is accessible:

```bash
gcloud storage buckets list
```

Target bucket:
```text
gs://gmajor_11
```

---

### 3.3 Download Bucket to Local VM

Create a local backup directory:
```bash
mkdir -p /home/thoa-duong/Documents/Backup_gcs/gmajor_11
```

Sync data from GCS to local storage:
```bash
gcloud storage rsync -m -r gs://gmajor_11 /home/thoa-duong/Documents/Backup_gcs/gmajor_11
```

**Notes:**
- `-m` enables parallel (multi-threaded) transfers
- `-r` syncs directories recursively

---

## 4. Restore Google Cloud Storage (New GCP Account)

### 4.1 Login to New GCP Account
Authenticate with the new GCP account:

```bash
gcloud auth login
```

(Optional) Set new project:
```bash
gcloud config set project NEW_PROJECT_ID
```

---

### 4.2 Verify Destination Bucket
Ensure the destination bucket exists:

```bash
gcloud storage buckets list
```

If the bucket does not exist, create it:
```bash
gcloud storage buckets create gs://gmajor_11 --location=asia-southeast1
```

---

### 4.3 Upload Local Backup to New Bucket

Sync local data back to GCS:
```bash
gcloud storage rsync -m -r /home/thoa-duong/Documents/Backup_gcs/gmajor_11 gs://gmajor_11
```

---

## 5. Verification

After restore, verify data integrity:

```bash
gcloud storage ls gs://gmajor_11
```

(Optional) Compare file count:
```bash
ls -R /home/thoa-duong/Documents/Backup_gcs/gmajor_11 | wc -l
gcloud storage ls -r gs://gmajor_11 | wc -l
```

---

## 6. Common Issues & Notes

- **Permission denied**  
  Ensure the account has:
  - `Storage Object Viewer` (read)
  - `Storage Object Admin` (write)

- **Uniform Bucket-Level Access error**  
  Check bucket IAM settings before syncing.

- **Large buckets**  
  Make sure VM disk size is sufficient.

---

## 7. References

- Google Cloud Storage Console:  
  https://console.cloud.google.com/storage/overview

- GCloud Storage CLI Docs:  
  https://cloud.google.com/sdk/gcloud/reference/storage

---

✅ Backup & restore completed successfully.
