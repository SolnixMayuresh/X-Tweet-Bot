# 🤖 AI-Powered Twitter Automation (n8n + Airtable)

This project is an **end-to-end automated Twitter posting system** built using **n8n**, **Airtable**, **Google Drive**, and **AI (LLM + Embeddings)**.

It automatically:

* Reads topics from Airtable
* Generates tweets using AI
* Posts them to Twitter (X)
* Updates Airtable with status & usage date
* Handles failures gracefully with fallback logic

---

## 🧱 Tech Stack

* **n8n** – Workflow automation
* **Airtable** – Topic & tweet database
* **Google Drive** – Source file storage
* **AI Agent (LLM)** – Tweet generation
* **Embeddings (Google Gemini)** – Context understanding
* **Twitter/X API** – Tweet publishing

---

## 📊 Airtable Structure

Your Airtable base should contain the following columns:

| Column Name           | Type          | Description                  |
| --------------------- | ------------- | ---------------------------- |
| **Topic**             | Text          | Main topic for the tweet     |
| **Topic description** | Long text     | Extra context/details for AI |
| **Status**            | Single select | `to do`, `used`, `failed`    |
| **tweet**             | Long text     | Generated tweet content      |
| **used date**         | Date          | Date when tweet was posted   |

---

## 🔄 Workflow Overview

### 1️⃣ Download & Prepare Data (Google Drive → Airtable)

* Triggers when a **PDF file** is added to the **main Google Drive folder**
* ✅ **Only PDF format is supported** (`.pdf`)
* Downloads the file
* Extracts text content from the PDF
* Cleans & formats text
* Creates or updates records in Airtable
* **Moves the processed file from the main folder to a separate archive/processed folder**

📌 This ensures:

* The same file is **never read twice**
* No duplicate Airtable records are created
* Clean separation between *new* and *processed* files

❗ Non-PDF files are ignored or routed to error handling (recommended).

---

### 2️⃣ Search Record for Processing

* Scheduled trigger runs at a fixed interval
* Searches Airtable for records where:

  * `Status = to do`

---

### 3️⃣ Process One by One

* Uses **Limit** node to process only **one record at a time**
* Prevents spam and API rate-limit issues

---

### 4️⃣ Generate Tweet (Primary AI Agent)

AI Agent receives:

* Topic
* Topic description
* Optional context from embeddings

AI generates:

* Concise
* Platform-optimized
* Engaging tweet text

---

### 5️⃣ Fallback Tweet Generator (If AI Fails)

If the main AI agent fails:

* Secondary AI agent is triggered
* Generates a simplified tweet
* Prevents workflow failure

---

### 6️⃣ Upload Tweet to Twitter (X)

* Uses Twitter/X node
* Posts the generated tweet

---

### 7️⃣ Clean & Update Airtable

After successful posting:

* Cleans tweet text
* Updates Airtable record:

  * `tweet` → generated content
  * `Status` → `used`
  * `used date` → current date

---

### 8️⃣ Error Handling

If all tweet generation attempts fail:

* Workflow stops
* Record can be marked as `failed`
* Prevents infinite loops

---

## 🚨 Global Error Handling

This workflow includes **global error handling** to ensure stability and prevent silent failures.

**How it works:**

* Any unhandled error in any node is captured by the **Global Error Workflow**
* Errors are logged with context (node name, execution, message)
* The main workflow is safely stopped to avoid partial or duplicate processing

**Benefits:**

* Centralized error visibility
* Easier debugging and monitoring
* Prevents infinite loops or repeated failures

📌 Recommended actions inside the global error handler:

* Log error details (execution ID, workflow name)
* Mark related Airtable records as `failed` (optional)
* Send notifications (Slack / Email / Discord)

---

## 🧠 AI & Embeddings Usage

* **Embeddings** are used to:

  * Improve topic understanding
  * Maintain consistency in tone
  * Avoid repetitive content

* **LLM Agent** focuses on:

  * Short-form content
  * Twitter/X best practices
  * Clear call-to-action when needed

---

## ⚙️ Environment Variables Required

### 🕒 Timezone Configuration

To ensure correct scheduling and accurate `used date` updates, **set the n8n timezone to Asia/Kolkata**.

You can do this in one of the following ways:

**Option 1: n8n Environment Variable**

```
GENERIC_TIMEZONE=Asia/Kolkata
```

**Option 2: n8n UI**

* Go to **Settings → General**
* Set **Timezone** to `Asia/Kolkata`

This ensures:

* Scheduled triggers run in IST
* Airtable `used date` matches local time
* No mismatch in tweet posting time

---

## ⚙️ Environment Variables Required

Set these in your n8n environment:

* `AIRTABLE_API_KEY`
* `AIRTABLE_BASE_ID`
* `TWITTER_API_KEY`
* `TWITTER_API_SECRET`
* `TWITTER_ACCESS_TOKEN`
* `TWITTER_ACCESS_SECRET`
* `GOOGLE_API_KEY` (for Gemini / Embeddings)

---

## 🚀 How to Use

1. Import the workflow into n8n
2. Configure credentials (Airtable, Google, Twitter, AI)
3. Prepare Airtable with correct columns
4. Add topics with `Status = to do`
5. Activate the workflow

Tweets will now be:
👉 generated automatically
👉 posted to Twitter
👉 tracked inside Airtable

---

## ✅ Best Practices

* Keep topics short & clear
* Avoid adding too many pending records at once
* Use embeddings for better tweet quality
* Monitor failed records periodically

---

## 📌 Future Improvements (Optional)

* Hashtag generation
* Thread (multi-tweet) support
* Multiple Twitter accounts
* Analytics & engagement tracking

---

## 🧑‍💻 Author

**Mayuresh Gangankar**
Built with ❤️ using **n8n + AI automation**

---
