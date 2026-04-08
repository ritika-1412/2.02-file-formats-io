# 04 — File Formats and Data I/O

**Course:** Python for AI & Data  
**Unit:** 1 — Applying Python Fundamentals to Solve Data Problems  
**Session Time:** 60 minutes

---

## Prerequisites

- Modules 1–3 complete
- Comfortable with functions, lists, and dictionaries

---

## Table of Contents

1. [Module Overview](#module-overview)
2. [Learning Objectives](#learning-objectives)
3. [Session Breakdown](#session-breakdown)
4. [Why File I/O Matters](#why-file-io-matters)
5. [Working with CSV Files](#working-with-csv-files)
6. [Working with JSON Files](#working-with-json-files)
7. [File Paths with pathlib](#file-paths-with-pathlib)
8. [Reproducible Project Structure](#reproducible-project-structure)
9. [Wrap-Up](#wrap-up)
10. [Resources](#resources)

---

## Module Overview

In Labs 1–3 you hardcoded BeanScene's data directly in your notebooks. That works for small examples, but in the real world data lives in files — CSVs from a sales system, JSON from an API, exports from a spreadsheet. A data analyst's job almost always starts with reading a file and ends with writing one.

In this module you'll learn how to move data **into and out of Python** using the two most common formats in analytics: **CSV** and **JSON**. You'll also learn how to handle file paths reliably using Python's `pathlib` library so your code works on any machine.

---

## Learning Objectives

By the end of this module, you'll be able to:

- Read and write CSV files using Python's `csv` module
- Read and write JSON files using Python's `json` module
- Handle file paths reliably using `pathlib.Path`
- Separate raw data from processed outputs in a reproducible project structure
- Write data I/O functions that are reusable and portable

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Why File I/O Matters | 5 min |
| 2 | Working with CSV Files | 20 min |
| 3 | Working with JSON Files | 15 min |
| 4 | File Paths with pathlib | 10 min |
| 5 | Reproducible Project Structure | 5 min |
| 6 | Wrap-Up | 5 min |

---

## Why File I/O Matters

In every data and AI project, the first thing you do is **read data from somewhere** and the last thing you do is **write results somewhere**. Everything in between — cleaning, analysing, modelling — depends on getting these two steps right.

**Why this matters in practice:**
- A colleague sends you a CSV export from the sales system — you need to load it
- Your analysis results need to be saved so the next analyst can pick up where you left off
- A web API returns JSON — you need to parse it into Python objects
- Your model's predictions need to be written to a file for the dashboard team

Getting file I/O right also means your work is **reproducible**: anyone with the same files can run your notebook and get the same results.

---

## Working with CSV Files

CSV (Comma-Separated Values) is the most common format for tabular data. Every spreadsheet application can read and write it, and it is human-readable.

### Reading a CSV — `csv.DictReader`

`csv.DictReader` reads each row as a dictionary, using the header row as keys. This is almost always what you want.

```python
import csv

with open("data/raw/beanscene_menu.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    menu = list(reader)

# Each row is a dictionary
print(menu[0])
# {'name': 'Espresso', 'price': '3.50', 'units_sold': '42', 'category': 'Hot Drinks'}
```

**What to notice:**
- `newline=""` is required for correct cross-platform behaviour — always include it
- `encoding="utf-8"` is best practice — always specify encoding
- Values come in as **strings** — even numbers. You must convert them explicitly.

---

### Type Conversion After Loading

```python
# Values are strings by default — convert before calculating
row = menu[0]
print(type(row["price"]))      # <class 'str'>
print(type(row["units_sold"])) # <class 'str'>

# Convert when you need to calculate
price = float(row["price"])
units = int(row["units_sold"])
revenue = price * units
print(f"Revenue: ${revenue:.2f}")
```

---

### Demo: Load and Process the Full Menu

```python
import csv

menu = []
with open("data/raw/beanscene_menu.csv", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        menu.append({
            "name": row["name"],
            "price": float(row["price"]),
            "units_sold": int(row["units_sold"]),
            "category": row["category"],
        })

# Inspect
for item in menu:
    revenue = item["price"] * item["units_sold"]
    print(f"{item['name']:<14}: ${revenue:.2f}")
```

---

### Writing a CSV — `csv.DictWriter`

```python
import csv

# Results we want to save
results = [
    {"name": "Espresso",  "revenue": 147.00, "tier": "Low"},
    {"name": "Latte",     "revenue": 289.75, "tier": "High"},
]

with open("data/processed/menu_results.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "revenue", "tier"])
    writer.writeheader()
    writer.writerows(results)

print("Saved successfully")
```

**Key rules:**
- Always call `writer.writeheader()` first
- `fieldnames` controls column order
- The `"w"` mode overwrites an existing file — use `"a"` to append

---

## Working with JSON Files

JSON (JavaScript Object Notation) is the standard format for structured, nested data — especially data from APIs and configuration files. Python's built-in `json` module handles it directly.

### Reading a JSON File

```python
import json

with open("data/raw/beanscene_config.json", encoding="utf-8") as f:
    config = json.load(f)

print(config)
print(type(config))                              # <class 'dict'>
print(config["cafe_name"])                       # BeanScene
print(config["performance_thresholds"]["high"])  # 250.0
```

**What to notice:**
- `json.load()` parses the file into native Python objects — dicts, lists, strings, numbers, booleans
- JSON objects become Python `dict`
- JSON arrays become Python `list`
- JSON numbers become Python `int` or `float` — no manual conversion needed

---

### Demo: Use Config to Drive Analysis

```python
import json, csv

with open("data/raw/beanscene_config.json", encoding="utf-8") as f:
    config = json.load(f)

high_threshold = config["performance_thresholds"]["high"]
low_threshold = config["low_performer_threshold"]

menu = []
with open("data/raw/beanscene_menu.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        menu.append({
            "name": row["name"],
            "price": float(row["price"]),
            "units_sold": int(row["units_sold"]),
        })

for item in menu:
    revenue = item["price"] * item["units_sold"]
    if revenue >= high_threshold:
        tier = "High"
    elif revenue >= low_threshold:
        tier = "Medium"
    else:
        tier = "Low"
    print(f"{item['name']:<14}: ${revenue:.2f}  [{tier}]")
```

> **Pattern to remember:** Load your configuration from JSON, load your data from CSV, then drive your analysis using the config values. This means changing thresholds never requires editing code — only the config file.

---

### Writing a JSON File

```python
import json

summary = {
    "cafe_name": "BeanScene",
    "week": "2024-W04",
    "total_revenue": 1521.75,
    "best_seller": "Latte",
    "items_analysed": 8,
}

with open("data/processed/weekly_summary.json", "w", encoding="utf-8") as f:
    json.dump(summary, f, indent=2)

print("Summary saved")
```

`indent=2` makes the output human-readable — always use it when writing JSON.

---

## File Paths with pathlib

Hardcoding file paths as strings (`"../data/raw/file.csv"`) is fragile — it breaks when the project is moved or run from a different directory. Python's `pathlib.Path` solves this.

### Demo: Building Paths Reliably

```python
from pathlib import Path

# Define paths relative to the project root
DATA_DIR = Path("data")
RAW_DIR = DATA_DIR / "raw"
PROCESSED_DIR = DATA_DIR / "processed"

# Build a file path
menu_path = RAW_DIR / "beanscene_menu.csv"
print(menu_path)            # data/raw/beanscene_menu.csv
print(menu_path.exists())   # True (if the file is there)
```

**What to notice:**
- `/` between `Path` objects builds paths — no string concatenation needed
- `.exists()` lets you verify a file is there before trying to open it
- Path objects work seamlessly with `open()` — pass them directly

---

### Creating Directories Programmatically

```python
# Create the processed folder if it doesn't exist
PROCESSED_DIR.mkdir(parents=True, exist_ok=True)
```

`exist_ok=True` means "don't raise an error if the folder already exists" — safe to call every time.

---

### Confirm Your Working Directory

When you launch JupyterLab from the repo root and work in `notebooks/`, the working directory is the repo root. Confirm it at the top of every notebook:

```python
import os
print("Working directory:", os.getcwd())
# Should show: .../04-file-formats-and-data-io
```

If it doesn't match, either re-launch JupyterLab from the correct folder, or adjust your paths accordingly.

---

## Reproducible Project Structure

Professional data projects always separate raw data from processed outputs and keep code in a dedicated folder. This is the standard structure you will follow in all labs from this point forward:

```text
your-project/
│
├── data/
│   ├── raw/          ← original source files — never modify these
│   └── processed/    ← outputs your code produces — can always be regenerated
│
├── notebooks/        ← your Jupyter notebooks
│
├── requirements.txt  ← library versions for reproducibility
│
└── README.md         ← describes the project
```

**Why this matters:**
- Raw data is sacred — if you accidentally overwrite it, you lose the source of truth
- Processed outputs can always be regenerated by running the notebook from scratch
- Anyone cloning your repo immediately knows where inputs are and where outputs go
- This structure is what real data teams use — building the habit now pays off throughout your career

---

## Wrap-Up

Reflect on these questions before moving into the lab:

- What is the difference between `csv.reader` and `csv.DictReader`?
- Why do numeric values come in as strings when you read a CSV?
- What is the advantage of storing thresholds in a JSON config file instead of hardcoding them?
- Why should raw data never be modified?

---

## Resources

- **Python csv module:** https://docs.python.org/3.11/library/csv.html
- **Python json module:** https://docs.python.org/3.11/library/json.html
- **pathlib:** https://docs.python.org/3.11/library/pathlib.html
- **Real Python — Reading and Writing CSV:** https://realpython.com/python-csv/
- **Real Python — Working with JSON:** https://realpython.com/python-json/
