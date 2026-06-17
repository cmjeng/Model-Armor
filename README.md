# GCP Model Armor Batch Inspection Tools

This repository contains two Python scripts designed to perform batch security inspections and safety reviews on text prompts. They utilize Google Cloud Vertex AI (Gemini 2.5 Flash) and the Model Armor REST API to detect Responsible AI (RAI) violations, Prompt Injection (PI) attacks, and other security risks.

---

## 🛠 Prerequisites & Authentication

These scripts require a Google Cloud Project with the appropriate APIs enabled (Vertex AI API and Model Armor API) and utilize Application Default Credentials (ADC) for secure authentication.

### 1. Install Google Cloud CLI

If you haven't already, install the [Google Cloud CLI](https://cloud.google.com/sdk/docs/install).

### 2. Login via Application Default Credentials (ADC)

You must authenticate your local environment so the Python scripts can securely connect to Google Cloud services. Open your terminal or command prompt and run:

```bash
gcloud auth application-default login

```

*This command will open a browser window. Log in with the Google account that has access to your target Google Cloud project. A local credentials file will be generated.*

### 3. Set Project Quota (Optional but Recommended)

To ensure quota billing is attributed to the correct project, you can set your active project:

```bash
gcloud config set project YOUR_PROJECT_ID
gcloud auth application-default set-quota-project YOUR_PROJECT_ID

```

---

## 📦 Environment Setup

Ensure you have Python 3.x installed. Install the required Python dependencies using `pip`.

```bash
pip install google-genai google-auth requests

```

* `google-genai`: The official Google GenAI SDK required for Vertex AI Gemini interactions.
* `google-auth`: Handles the ADC authentication lifecycle.
* `requests`: Used to make RESTful HTTP calls to the Model Armor API.

---

## 📜 Script Usage

### Tool 1: Model Armor Batch Inspection (`model-armor-batch-inspection.py`)

This script interacts directly with the Google Cloud Model Armor `sanitizeUserPrompt` REST API. It evaluates inputs against predefined security templates to check for CSAM, Malicious URIs, RAI violations, PI/Jailbreaks, and Sensitive Data Protection (SDP) triggers.

**Configuration:**
Before running, open `model-armor-batch-inspection.py` and update the following variables:

* `PROJECT_ID = 'YOUR_PROJECT_ID'`
* `LOCATION = 'us-central1'`
* `TEMPLATE_NAME = 'MODEL_ARMOR_TEMPLATE_NAME'` (The name of your pre-configured Model Armor template)
* `INPUT_CSV = 'Test_Prompt.csv'` (Your source file with a header named `Test Prompt`)

**Execution:**

```bash
python model-armor-batch-inspection.py

```

**Output:** Generates `Test_Prompt_Result.csv` appending:

1. **Match (Yes, No):** Indicates if any Model Armor filter was triggered.
2. **Matched Types:** A comma-separated list of the specific filters triggered (e.g., `rai_dangerous`, `pi_and_jailbreak`, `sdp(LOCATION)`).

---

### Tool 2: Gemini RAI & PI Reviewer (`Review_Model_Armor_Results.py`)

This script uses `gemini-2.5-flash` to act as a security expert. It reads a batch of prompts, evaluates them for toxicity, Responsible AI violations, and Prompt Injections, and translates the prompts into Traditional Chinese for human review.

**Key Features:**

* Bypasses default Gemini safety filters (`BLOCK_NONE`) to ensure the model analyzes toxic content instead of blocking the request and throwing an error.
* Outputs structured JSON containing a strict "Safety" / "Toxic" classification and the translated text.

**Configuration:**
Before running, open `Review_Model_Armor_Results.py` and update the following variables:

* `PROJECT_ID = "YOUR_PROJECT_ID"`
* `LOCATION = "us-central1"` (Ensure your region supports the selected model)
* `INPUT_FILE = "PI_RAI_Content.csv"` (Your source file containing prompts in the first column)

**Execution:**

```bash
python Review_Model_Armor_Results.py

```

**Output:** Generates `PI_RAI_Content_Reviewed_Full.csv` appending the "Gemini Review Comments" and "中文語意" (Chinese translation) columns.
