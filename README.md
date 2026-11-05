# Smart Excel Stream & Validator (ODC)

**Validates, cleans, and parses massive Excel files at runtime using rule-driven JSON configuration. Designed for high performance via C# streaming to eliminate timeouts and payload errors in OutSystems Developer Cloud.**

---

## 🎯 Why choose this component?

Processing massive Excel files (with over 50,000 or 100,000 rows) directly within OutSystems visual flows traditionally causes three major issues: **Server timeouts**, **high RAM consumption**, and **payload size errors (`OS-FLRT-60009` when exceeding the 5.5MB platform limit)**.

**Smart Excel Stream & Validator** solves this natively:
* **Extreme Performance:** Processes 100,000 rows in less than 4 seconds using *C# Streaming* (.NET Core).
* **Smart Memory Consumption:** Reads the file row by row, without loading the entire binary into the container's RAM.
* **Fail-Safe Architecture (Chunking):** Implements native pagination within the C# logic to slice the JSON output, bypassing the strict 5.5MB ODC platform limit.
* **Dynamic Validation (Rule-Driven):** No need to build dozens of visual `If` nodes. Define your validation rules in a simple JSON schema and let the engine do the heavy lifting.

---

## 🏗️ API Reference (Server Action Signature)

The component exposes a single centralized Server Action: **`ValidateExcelFile`**.

### Input Parameters
* **`FileBinary`** (*Binary Data*, Mandatory): The binary content of the Excel file (`.xlsx`) uploaded by the user.
* **`ValidationRulesJSON`** (*Text*, Mandatory): A JSON array defining the validation rules and constraints for each column.
* **`PageNumber`** (*Integer*, Mandatory): The page/batch number you want to read (1-based index, e.g., `1` for the first batch).
* **`PageSize`** (*Integer*, Mandatory): The batch size of rows you want to receive (Recommended: `5000` rows per batch to keep the payload lightweight).

### Output Parameters
* **`CleanedDataJSON`** (*Text*): A JSON array containing the sanitized and cleaned row data belonging only to the requested page. Ready to be deserialized in OutSystems.
* **`ErrorListJSON`** (*Text*): A structured JSON list containing all validation errors found in the processed rows (includes row number, column name, invalid value, and error message).
* **`IsValid`** (*Boolean*): Returns `True` if the processed batch meets all specified rules; returns `False` if structural errors or validation failures are found.

---

## ⚙️ Validation Rules Configuration (Schema)

The engine accepts a direct array of JSON objects. Each object represents a column you want to validate in the Excel file.

### Supported Format
```json
[
  {
    "column": "Name",
    "required": true
  },
  {
    "column": "Age",
    "min": 18
  }
]
```

### Available Properties:
* **`column`**: The exact name of the column header in the first row of the Excel file (*Case-Sensitive*).
* **`required`**: If set to `true`, rejects the row if the cell is empty.
* **`min`**: Defines the minimum accepted value for numeric data.

---

## 🚀 Implementation Guide (Step-by-Step)

To implement this component in your Reactive Screen, follow these 4 simple steps:

### Step 1: File Capture
1. Drag the **Upload** widget onto your screen.
2. Bind the *File Content* and *File Name* properties to two local variables (`UploadedContent` and `UploadedName`).
3. Set the Upload widget's *Accept* property to `.xlsx`.

### Step 2: Create Structures for Deserialization
Create the following **Structures** in your OutSystems module to receive the sanitized JSON data:
* **`ExcelRowResult`**: Add attributes that match your Excel columns (e.g., `Name` as *Text* and `Age` as *Integer*).
* **`ExcelValidationError`**: Attributes to catch errors: `Row` (Integer), `Column` (Text), `Value` (Text), `Error` (Text).

### Step 3: Configure the Logical Flow (Screen Action)
On your processing button's click event, build the following flow:
1. Call the **`ValidateExcelFile`** Server Action. Pass the binary file, the rules JSON, the current page (e.g., `1`), and the batch size (e.g., `5000`).
2. Add a **JSON Deserialize** node pointing to the `CleanedDataJSON` output and set the Data Type to `ExcelRowResult List`.
3. Assign the Deserialize output to a local list variable (`CleanedRowsList`) to feed your screen.

### Step 4: Interface Pagination
For giant files, use a loop or the default pagination of the OutSystems **Table** widget. Whenever the user changes pages on the table, re-execute the Screen Action incrementing the **`PageNumber`** parameter (e.g., Page 1, Page 2, Page 3) until `CleanedDataJSON` returns an empty array, indicating the end of the document.

---

## 📈 Best Practices & Limitations
* **Default Sheet:** The engine automatically reads the **first active Worksheet (Tab)** inside the Excel file.
* **Headers:** The first row of the Excel file is strictly treated as the header row. Make sure there are no blank rows at the top of the document.
* **Maximum Batch Size:** Although the C# engine can process larger slices, avoid setting a `PageSize` higher than `10000` rows to ensure instant data transfer without network latency.
