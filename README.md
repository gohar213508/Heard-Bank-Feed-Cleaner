# Financial Data Feed Automation Engine

An enterprise-grade, high-performance C# .NET Core background processing service designed for financial data feed automation. The application ingests bank feed CSV files, performs strict row-level validations, isolates malformed records, logs valid transactions directly into Microsoft SQL Server, and archives processed files.

---

## 🛠 Project Features

- **Automated Ingestion**: Monitored directory processing for incoming bank CSV feeds.
- **Data Hygiene & Validation**:
  - Validates IBAN structure and length.
  - Normalizes amounts and formats timestamps into ISO standards.
  - Runtime deduplication checks on unique `TxnId`s.
- **Isolated Error Handling**: Malformed rows are routed directly to audit tables without stopping the execution pipeline.
- **Relational Storage**: Stores summary metrics, cleaned records, and audit error trails across normalized SQL Server tables.
- **Resilient Fallback**: In-memory and local file logging fallback in case of temporary database connectivity loss.
- **Automated Archiving**: Prevents duplicate execution by moving processed files to an archive directory with precise timestamps.

## 🚀 First-Time Setup & Execution Guide

Follow these steps sequentially to set up, configure, and execute the application for the first time.

### Step 1: Database Setup

Before running the application, create the database schema in your Microsoft SQL Server instance by running the script below:

```sql
-- 1. Summary Level Metrics Table
CREATE TABLE FileProcessingSummary (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    FileName NVARCHAR(255) NOT NULL,
    TotalRows INT NOT NULL,
    ValidRows INT NOT NULL,
    RejectedRows INT NOT NULL,
    Status VARCHAR(50) NOT NULL,
    ProcessedAt DATETIME DEFAULT GETUTCDATE()
);

-- 2. Cleaned Output Records Table
CREATE TABLE CleanedTransactions (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    FileName NVARCHAR(255) NOT NULL,
    TxnId VARCHAR(20) NOT NULL,
    TxnDate VARCHAR(10) NOT NULL,
    AccountNo VARCHAR(34) NOT NULL,
    Amount DECIMAL(18,2) NOT NULL,
    Currency VARCHAR(3) NOT NULL,
    Type VARCHAR(2) NOT NULL,
    Narration NVARCHAR(100) NULL,
    CreatedAt DATETIME DEFAULT GETUTCDATE()
);

-- 3. Rejected Lines Audit Table
CREATE TABLE RejectedRowLogs (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    FileName NVARCHAR(255) NOT NULL,
    LineNumber INT NOT NULL,
    RawLine NVARCHAR(MAX) NOT NULL,
    Reasons NVARCHAR(MAX) NOT NULL,
    CreatedAt DATETIME DEFAULT GETUTCDATE()
);






APPsetting 

{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=FinancialFeedDb;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "FileStorage": {
    "InputFolderPath": "./Data/Input",
    "OutputFolderPath": "./Data/Output",
    "ErrorFolderPath": "./Data/Errors",
    "ArchiveFolderPath": "./Data/Archive"
  }
}


-- View overall processing summaries and statistics
SELECT * FROM FileProcessingSummary;

-- View successfully cleaned and ingested records
SELECT * FROM CleanedTransactions;

-- Audit rejected rows along with specific parsing error reasons
SELECT * FROM RejectedRowLogs;


## Part B: Database Flexibility & Resilience Architecture

### 1. Multi-Database Provider Support (Interface & DI)
Application follows the **Open/Closed Principle** using the `IProcessingLogStore` interface. The database provider can be dynamically switched without recompiling the application binary.

* **Supported Providers**: SQL Server and Oracle.
* **Configuration**: Change the `"LogProvider"` value in `appsettings.json`:
  ```json
  "LogProvider": "SqlServer" // Options: "SqlServer" | "Oracle"




# Heard-Bank-Feed-Cleaner
