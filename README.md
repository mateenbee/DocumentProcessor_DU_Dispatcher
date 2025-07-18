# 📄 DocumentProcessor_DU_Dispatcher (UiPath REFramework)

> ⚠️ **Note**: This is a **template** dispatcher process built on UiPath's REFramework to support document understanding workflows. It is designed for general use and should be adapted to match specific project and environment requirements.

---

## 🚀 Overview

The **DocumentProcessor_DU_Dispatcher** is a UiPath Dispatcher that prepares documents for a Document Understanding (DU) Performer process. It monitors a shared folder where various document types (e.g., birth certificates, marriage certificates, bank statements, utility bills, etc.) are dropped.

Its job is to:
- Detect and collect new files.
- Extract metadata (like Customer ID) from the file name.
- Add each document as a transaction item in an Orchestrator queue.
- Organize files into structured folders to prevent reprocessing.

---

## 📁 Document Flow

SharedLocation/
└── 2025-07-18/
└── Input/
├── Unprocessed/ <-- Files dropped by external systems
└── Processed/ <-- Files that have been queued
└── Retry/ <-- Files that failed earlier and need reprocessing

---

## 🧠 How It Works

### 🔧 Initialization State
- Checks if today's folder structure exists (`yyyy-MM-dd/Input/Unprocessed`, etc.).
- If not present, the bot **creates the folder hierarchy**.
- Prepares environment for transaction processing.

### 📥 Get Transaction Data State
- Checks if any files exist in the **Retry folder** (high priority).
- If no retries exist, it pulls new files from: Today's Folder → Input → Unprocessed
- Builds an **array of file paths** (strings) to be used as transaction items.

### ⚙️ Process Transaction State
- For each file:
- Extracts key metadata from the **filename**, such as:
  - `CustomerID`
  - `Original File Name`
  - `Received Date/Time` (uses `Now`)
- Uses the **filename as the queue item reference**.
- Adds a **queue item** to Orchestrator with required metadata.
- Moves the file from **Unprocessed → Processed** to prevent duplicate processing.

---

## 📑 Filename Format Assumption

The dispatcher expects filenames to follow a consistent pattern, such as: 12345_BankStatement_JohnDoe.pdf

From this, the bot may extract:
- `CustomerID`: 12345
- `DocumentType`: BankStatement
- `OriginalFileName`: JohnDoe.pdf
- `ReceivedTimestamp`: current system time

> 🔁 You can customize the filename parsing logic in the `Process.xaml` file.

---

## 🧩 Queue Item Schema

Each queue item contains:
| Field             | Description                        |
|------------------|------------------------------------|
| `Reference`       | File name (used to ensure uniqueness) |
| `CustomerID`      | Extracted from file name           |
| `OriginalFileName`| Original document file name        |
| `ReceivedDate`    | Timestamp when file was picked up  |
| `FilePath`        | Full path to the local file        |

---

---

## ⚙️ Configuration Settings

In `Data\Config.xlsx`:

| Name                 | Description                                |
|----------------------|--------------------------------------------|
| `InputFolderPath`    | Root folder path for daily document intake |
| `QueueName`          | Orchestrator queue for DU transactions     |
| `RetryFolderPath`    | Location of retry files                    |
| `ProcessedFolderPath`| Where successfully queued files are moved  |

---

## 📅 Scheduling & Execution

- This Dispatcher should be scheduled to run  **before** the Performer bot.
- Make sure external systems drop files into the **Unprocessed** folder ahead of time.

---

## 🔒 Notes & Best Practices

- Ensure filename format is consistent across all document sources.
- Maintain proper **folder access permissions** for the bot.
- Implement error logging and alerts for unrecognized file formats.
- Queue reference (file name) ensures **idempotency** (no duplicate items).
- Retry folder logic allows for re-attempting failed files in a controlled way.

---

## 🔧 Future Improvements

- Add file format validation (PDF, TIFF, DOCX, etc.).
- Auto-archive processed files to external storage.
- Add email or Teams alert when invalid files are detected.
- Extend support for different naming conventions per document type.



