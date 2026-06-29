# Google Cloud Model Armor Custom Monitoring Dashboard

This repository contains the configuration file and deployment guide for the **Model Armor Custom Dashboard**. This dashboard leverages Google Cloud Monitoring and BigQuery-powered Log Analytics to provide real-time, comprehensive visibility into your LLM security layers.

It aggregates and visualizes critical safety metrics, including Prompt Injection/Jailbreak mitigation, Responsible AI (RAI) rules, CSAM blocks, Malicious URIs, and Sensitive Data Protection (SDP) safety nets.

---

## 🛠️ Prerequisites

Before importing the dashboard, your Google Cloud project must be configured to generate Model Armor security logs, analyze them via SQL, and expose counter metrics for time-series charts.

### 1. Enable Model Armor Logging
Model Armor does not log evaluation details by default. You must activate logging to populate the data streams.
*   Navigate to **Google Cloud Console** > **Security** > **Model Armor**.
*   Select your active **Model Armor Templates** or enter **Floor Settings** depending on your operational architecture.
*   Ensure that the **Logging** option is toggled **ON** for your attached templates and application endpoints. This ensures that `SanitizeOperationLogEntry` records are published to Cloud Logging.

### 2. Upgrade the `_Default` Log Bucket to Log Analytics
The core table widgets use advanced SQL queries to parse unstructured JSON payloads and handle multiple overlapping security matches. This functionality requires BigQuery-powered Log Analytics.
*   Go to **Logging** > **Log Storage**.
*   Locate your `_Default` bucket (or the custom bucket routing your security logs).
*   If not already done, click **Upgrade to use Log Analytics** and confirm.

### 3. Create the Custom Log-Based Metric
The time-series line chart tracking threat trends requires a custom user-defined counter metric.
*   Go to **Logging** > **Log-based Metrics**.
*   Click **Create Metric** and set up the following parameters:
    *   **Metric Type**: Counter
    *   **Log-based Metric Name**: `model_armor_blocked_count`
    *   **Description**: Tracks requests flagged or blocked by Model Armor filters.
    *   **Filter Selection**: Paste the exact filter string below:
        ```query
        jsonPayload.@type="[type.googleapis.com/google.cloud.modelarmor.logging.v1.SanitizeOperationLogEntry](https://type.googleapis.com/google.cloud.modelarmor.logging.v1.SanitizeOperationLogEntry)"
        jsonPayload.sanitizationResult.filterMatchState="MATCH_FOUND"
        ```
*   Click **Create Metric** to finalize the creation of the metric descriptor path: `logging.googleapis.com/user/model_armor_blocked_count`.

---

## 🚀 Deployment Steps

### Step 1: Import the Dashboard Configuration
1. Locate the file named `Model Armor Custom Dashboard.json` in this repository.
2. In the Google Cloud Console, navigate to **Monitoring** > **Dashboards**.
3. Click the **Sample Library / Import** button at the top of the pane.
4. Click **Import**, upload the `Model Armor Custom Dashboard.json` file, and confirm.

### Step 2: Configure Project ID & Timezone inside the Cloud Console UI
Because Cloud Monitoring's analytical engine requires rigid data routing identifiers, you must configure the query variables directly within your newly imported console dashboard.

1. Open your imported **Model Armor Custom Dashboard** and click the **Pencil Icon** (Edit Mode) at the top right.
2. Locate the two SQL-driven table widgets: **"Model Armor Filters"** and **"Model Armor Validation (Max 1,000 records)"**.
3. Click **Edit** on the table widget to expose the Cloud Logging SQL editor.
4. **Update Project ID**: Locate the `FROM` statement near the bottom of the queries and swap out the placeholder workspace ID for your actual Google Cloud Project ID:
    ```sql
    FROM
      -- 👈 Replace 'nick-demo-326723' with YOUR active Google Cloud Project ID
      `your-actual-project-id.global._Default._AllLogs` AS log
    ```
5. **Update Timezone (For the Details Log Table)**: Inside the **"Model Armor Validation"** query, locate the Common Table Expression (CTE) configuration at the top and swap the timezone text anchor to your native tracking zone:
    ```sql
    WITH config AS (
      -- 👈 Replace 'Asia/Taipei' with your operational zone (e.g., 'UTC', 'America/New_York')
      SELECT 'Asia/Taipei' AS tz   
    )
    ```
6. Click **Apply** on each edited widget pane, then click **Save** at the top right of the primary dashboard dashboard array to preserve your modifications.

---

## 🔒 Security & Optimization Note
*   **Data Resolution**: The main inspection log incorporates a fallback layer (`COALESCE`) that automatically parses standard plain text input as well as automatically executing `FROM_BASE64` decoding functions for byte streams.
*   **IAM Prerequisites**: Users interacting with this board require standard project `roles/monitoring.viewer` permissions alongside log view privileges (`roles/logging.viewTraces` or direct BigQuery Log Analytics access roles) to evaluate data rows.
