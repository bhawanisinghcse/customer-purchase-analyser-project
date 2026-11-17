# Customer Purchase Pattern Analyzer

A command-line Python application for parsing, validating, analyzing, and visualizing customer transaction records from CSV files. The tool processes order logs, filters malformed and negative data, calculates monthly revenue trends and central tendency statistics, and outputs an annotated Matplotlib bar chart.

---

## Capabilities

- **CSV Ingestion & Header Detection**: Loads transaction files with automatic detection of present or omitted header rows.
- **Data Validation & Error Reporting**: Filters out records with non-numeric amounts, negative amounts, or blank required fields, printing row-indexed console warnings while keeping valid rows.
- **Chronological Monthly Aggregation**: Groups revenue across calendar months in true chronological order (`Jan` through `Dec`) and flags peak revenue months.
- **Order Value Statistics**: Computes arithmetic mean and median order values using NumPy array routines.
- **Category Share Analysis**: Ranks product categories by total expenditure and computes relative percentage contributions.
- **Chart Generation**: Generates an annotated bar chart (`monthly_spending.png`) with currency labels above each column, supporting both desktop and headless environments.
- **Built-in Test Suite**: Runs all four standard acceptance test cases via `python main.py --test` with automated status verification.

---

## Project Structure

```text
customer purchase pattern analyser/
├── main.py                     # CLI entry point, analysis pipeline, and test runner
├── orders.csv                  # Default transaction dataset (8 valid orders)
├── requirements.txt            # Python dependencies (pandas, numpy, matplotlib)
├── README.md                   # User guide and operational documentation
├── summary.md                  # Technical architecture and statistical report
│
├── test_case_1.csv             # Standard dataset for monthly aggregation
├── test_case_2.csv             # Boundary dataset with a single transaction row
├── test_case_2_no_header.csv   # Boundary dataset without CSV headers
├── test_case_3.csv             # Dataset with non-numeric (N/A) and negative values
├── test_case_4.csv             # Multi-category revenue distribution dataset
│
├── monthly_spending.png        # Generated 300 DPI Matplotlib bar chart
├── test_case_1_screenshot.png  # Test Case 1 verification screenshot
├── test_case_2_screenshot.png  # Test Case 2 verification screenshot
├── test_case_3_screenshot.png  # Test Case 3 verification screenshot
└── test_case_4_screenshot.png  # Test Case 4 verification screenshot
```

---

## Requirements & Setup

### Prerequisites
- Python 3.8 or later
- `pip` package manager

### Installation

1. Navigate to the project directory:
   ```bash
   cd "E:\customer purchase pattern analyser"
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

Dependencies installed:
- `pandas` (>= 2.0.0): CSV parsing and tabular grouping
- `numpy` (>= 1.24.0): Vector operations, mean, and median calculations
- `matplotlib` (>= 3.7.0): Chart rendering and image export

---

## Usage

### 1. Interactive Mode
Run the script without arguments to start the interactive prompt:
```bash
python main.py
```
When prompted, press **Enter** to load `orders.csv` or supply a custom file path:
```text
Enter CSV filename [press Enter for 'orders.csv']:
```

### 2. Direct File Input
Pass a CSV file directly as a positional argument:
```bash
python main.py test_case_1.csv
```

### 3. Automated Test Runner
Verify all four acceptance test cases non-interactively:
```bash
python main.py --test
```

To display chart popups during the test run, include `--show-chart`:
```bash
python main.py --test --show-chart
```

### 4. Headless Execution
In automated environments without a display server (e.g., CI/CD or remote servers), set `HEADLESS=1` to save `monthly_spending.png` without opening a GUI window:
```bash
HEADLESS=1 python main.py --test
```

---

## Menu Options

When running interactively, the program presents six options:

```text
===========================================================
            CUSTOMER PURCHASE PATTERN ANALYZER
===========================================================

1. Monthly spending
2. Order statistics
3. Category share and statistics
4. Run all 4 official test cases (Acceptance Verification)
5. Load another CSV file
6. Exit
```

| Option | Command | Behavior |
| :--- | :--- | :--- |
| **1** | `Monthly spending` | Displays chronological monthly totals, overall total spending, identifies the top-performing month, and generates `monthly_spending.png`. |
| **2** | `Order statistics` | Outputs total valid order count, mean order value, median order value, and best month. |
| **3** | `Category share and statistics` | Lists categories in descending order of revenue with percentage share, plus mean and median order values. |
| **4** | `Run all 4 official test cases` | Runs the four verification cases sequentially, confirming input handling and calculations. |
| **5** | `Load another CSV file` | Prompts for a new file path, validates schema and contents, and swaps the active dataset. |
| **6** | `Exit` | Terminates execution cleanly. |

---

## CSV Data Specification

The analyzer expects comma-separated transaction records with four attributes:

| Column | Type | Constraints | Handling |
| :--- | :--- | :--- | :--- |
| `OrderID` | String / Integer | Required | Preserved for record reference; defaults to 1-based index if omitted. |
| `Month` | String | Non-empty | Normalized to 3-letter title case (e.g., `january` or `jan` becomes `Jan`). Blank values cause the row to be skipped. |
| `Category` | String | Non-empty | Product classification string. Blank values cause the row to be skipped. |
| `Amount` | Numeric | $\ge 0$ | Must be a valid non-negative number. Strings (`N/A`), empty values, and negative numbers (`-500`) are skipped with a warning. |

### Header Detection
`detect_header()` inspects the first line for known column keywords (`orderid`, `month`, `category`, `amount`). If absent, the file is parsed as raw data using default column names.

---

## Acceptance Test Scenarios & Verification Proof

The application includes four acceptance test scenarios corresponding to the test cases.

### Test Case 1: Standard Input (Monthly Spending)
- **Input File**: `test_case_1.csv` (8 rows)
- **Scope**: Monthly chronological grouping, overall sum, best month detection, chart output.
- **Terminal Output**:
  ```text
  Jan : Rs. 5000
  Feb : Rs. 12500
  Mar : Rs. 5000
  Apr : Rs. 11500
  Total: Rs. 34000
  Best month: Feb (Rs. 12500)
  (bar chart is displayed)
  ```
- **Proof**:
  ![Test Case 1 Verification](test_case_1_screenshot.png)

---

### Test Case 2: Boundary Input (Single Transaction)
- **Input File**: `test_case_2.csv` (`1,Jan,Grocery,2000`)
- **Scope**: Statistical integrity on minimal sample size ($N=1$).
- **Terminal Output**:
  ```text
  Orders analysed: 1
  Mean order value: Rs. 2000.00
  Median order value: Rs. 2000.00
  Best month: Jan (Rs. 2000)
  ```
- **Proof**:
  ![Test Case 2 Verification](test_case_2_screenshot.png)

---

### Test Case 3: Invalid Input & Warning Handling
- **Input File**: `test_case_3.csv` (Row 9 contains `N/A`, Row 10 contains `-500`)
- **Scope**: Validation filtering, row warning messages, preservation of valid records.
- **Terminal Output**:
  ```text
  Warning: Row 9 skipped - Amount must be a number.
  Warning: Row 10 skipped - Amount cannot be negative.
  Valid orders analysed: 8
  ```
- **Proof**:
  ![Test Case 3 Verification](test_case_3_screenshot.png)

---

### Test Case 4: Category Share & Order Statistics
- **Input File**: `test_case_4.csv` (8 rows)
- **Scope**: Descending category aggregation, revenue percentages, NumPy mean and median.
- **Terminal Output**:
  ```text
  Electronics : Rs. 18000 (52.94%)
  Grocery : Rs. 11000 (32.35%)
  Clothing : Rs. 5000 (14.71%)
  Mean order value: Rs. 4250.00
  Median order value: Rs. 3000.00
  ```
- **Proof**:
  ![Test Case 4 Verification](test_case_4_screenshot.png)

---

## Visualization Specification

The bar chart generated by Menu Option 1 is saved to `monthly_spending.png`:
- **Dimensions**: $8 \times 5$ inches rendered at 300 DPI.
- **Color**: Navy `#2b5c8f` bars with 0.55 width.
- **Labels**: Direct currency values centered above each bar.
- **Axes**: X-axis lists calendar months chronologically; Y-axis indicates spending in Rupees (Rs.).
- **Memory Management**: All figures are explicitly closed via `plt.close()` to avoid GUI thread memory retention.

---

## Error Handling Details

1. **Non-Existent File**: Displays `Error: File '<filename>' was not found.` and returns to the menu or terminates without a stack trace.
2. **Empty File**: Catches 0-byte files with `Error: File '<filename>' is empty.`.
3. **Missing Headers**: Reports exact missing column names when a header row is invalid.
4. **Zero Total Spending**: Prevents `ZeroDivisionError` when computing category percentages if total spending equals zero.
5. **Month Name Variations**: Converts mixed case, uppercase, or full month names (`february`, `FEB`) to standard 3-letter representations.
