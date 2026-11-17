# Technical Project Summary: Customer Purchase Pattern Analyzer

A technical and analytical summary report detailing the system architecture, mathematical foundations, data hygiene methodology, empirical benchmark findings, and acceptance verification of the **Customer Purchase Pattern Analyzer**.

---

## 1. Executive Summary & Problem Scope

Understanding consumer spending patterns across chronological time horizons and product categories is foundational to inventory procurement, promotional planning, and revenue forecasting. Transactional data exported from enterprise POS registers and online checkout gateways is frequently dirty, containing missing fields, string corruptions, or negative amounts.

The **Customer Purchase Pattern Analyzer** is an end-to-end Python 3 analytical application engineered to:
1. Ingest customer order logs from CSV transaction feeds.
2. Filter corrupt, non-numeric, or negative amounts with row-specific console warnings.
3. Aggregate monthly expenditures in chronological calendar order (`Jan` through `Dec`).
4. Evaluate central tendency via arithmetic mean and median order values using vector **NumPy** routines.
5. Rank product categories by expenditure and calculate percentage revenue contribution.
6. Provide an automated acceptance test runner verifying normal, boundary, invalid, and category distribution scenarios.
7. Render publication-ready **Matplotlib** visualizations exported to `monthly_spending.png`.

---

## 2. Technical Architecture & Module Structure

The solution is implemented in [`main.py`](main.py) as a modular, standalone pipeline adhering strictly to separation of concerns:

```text
                                 [ Raw CSV File ]
                                        │
                                        ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ 1. INGESTION LAYER: load_orders() & detect_header()                          │
│    • Checks file existence and non-zero byte size                            │
│    • Dynamically inspects first row to detect presence of column headers     │
│    • Enforces mandatory columns: OrderID, Month, Category, Amount            │
└──────────────────────────────────────┬───────────────────────────────────────┘
                                       │
                                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ 2. DATA HYGIENE LAYER: validate_orders()                                     │
│    • Strips leading/trailing whitespace and Unicode BOM artifacts            │
│    • Normalizes month names to standard 3-letter title case (Jan, Feb, ...)  │
│    • Traps non-numeric amounts: Warning: Row X skipped - Amount must be a ... │
│    • Traps negative amounts:    Warning: Row X skipped - Amount cannot be ...│
│    • Returns sanitized DataFrame with typed numeric series                   │
└──────────────────────────────────────┬───────────────────────────────────────┘
                                       │
                                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ 3. ANALYTICAL & STATISTICAL ENGINE (NumPy & Pandas)                          │
│    • show_monthly_spending(): Chronological calendar grouping & peak month   │
│    • show_order_statistics(): np.mean() & np.median() calculations          │
│    • show_category_share_and_stats(): Category ranking & percentage shares   │
│    • run_acceptance_test_cases(): Automated verification of all 4 test cases │
└──────────────────────────────────────┬───────────────────────────────────────┘
                                       │
                                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ 4. PRESENTATION & VISUALIZATION LAYER                                        │
│    • plot_monthly_chart(): High-resolution 300 DPI Matplotlib bar chart      │
│    • Exports monthly_spending.png with numerical value annotations           │
│    • Interactive console menu loop (Options 1 to 6)                          │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Mathematical & Statistical Foundations (NumPy)

All central tendency and percentage share calculations are performed using vectorized operations on contiguous NumPy arrays:

### 3.1 Arithmetic Mean (Mean Order Value)
For a set of $N$ valid transaction amounts $\{a_1, a_2, \dots, a_N\}$:
$$\bar{a} = \frac{1}{N} \sum_{i=1}^{N} a_i$$

- **Implementation**: `np.mean(amounts)`
- **Significance**: Quantifies the overall ticket average per transaction.

### 3.2 Median Order Value
For transaction amounts sorted in ascending order $a_{(1)} \le a_{(2)} \le \dots \le a_{(N)}$:
$$\text{Median} = \begin{cases} 
a_{\left(\frac{N+1}{2}\right)} & \text{if } N \text{ is odd} \\ 
\frac{a_{\left(\frac{N}{2}\right)} + a_{\left(\frac{N}{2}+1\right)}}{2} & \text{if } N \text{ is even} 
\end{cases}$$

- **Implementation**: `np.median(amounts)`
- **Statistical Significance in Retail Analytics**:
  Commercial sales distributions typically exhibit strong positive (right-hand) skewness caused by occasional high-ticket orders (e.g., consumer electronics or enterprise hardware). While the arithmetic mean is sensitive to extreme values, the median represents the true typical customer expenditure, unaffected by high-value outliers.

### 3.3 Product Category Revenue Percentage Share
For each product category $k$ with aggregated expenditure $S_k$ and total expenditure $S_{\text{total}} = \sum_{j=1}^{M} S_j$:
$$\text{Percentage Share}_k = \left( \frac{S_k}{S_{\text{total}}} \right) \times 100\%$$

- **Division-by-Zero Guard**: If $S_{\text{total}} = 0$, the algorithm suppresses division errors and outputs an explanatory notification.

---

## 4. Empirical Benchmark Findings (`orders.csv`)

Analyzing the default benchmark dataset [`orders.csv`](orders.csv) yields the following empirical metrics:

### 4.1 Summary Statistics
| Metric | Quantitative Value | Computational Method |
| :--- | :--- | :--- |
| **Valid Orders Analyzed** | 8 | Filtered row count after sanitization |
| **Total Expenditure** | Rs. 34,000 | Vector sum: `np.sum(amounts)` |
| **Mean Order Value** | Rs. 4,250.00 | Arithmetic average: `np.mean(amounts)` |
| **Median Order Value** | Rs. 3,000.00 | Positional midpoint: `np.median(amounts)` |
| **Peak Spending Month** | February (Rs. 12,500) | Maximum monthly aggregation |

### 4.2 Chronological Monthly Spending Distribution
| Calendar Month | Total Spending | Order Count | Contributing Categories |
| :--- | :--- | :---: | :--- |
| **January** | Rs. 5,000 | 2 | Grocery (Rs. 2,000), Clothing (Rs. 3,000) |
| **February** | Rs. 12,500 | 2 | Grocery (Rs. 2,500), Electronics (Rs. 10,000) |
| **March** | Rs. 5,000 | 2 | Grocery (Rs. 3,000), Clothing (Rs. 2,000) |
| **April** | Rs. 11,500 | 2 | Electronics (Rs. 8,000), Grocery (Rs. 3,500) |
| **Total** | **Rs. 34,000** | **8** | — |

*Analytical Finding*: Monthly revenue is concentrated in February (Rs. 12,500) and April (Rs. 11,500), driven primarily by major Electronics purchases.

### 4.3 Category Revenue Contribution & Pareto Distribution
| Category | Total Revenue | Percentage Share | Cumulative Share | Order Frequency |
| :--- | :--- | :---: | :---: | :---: |
| **Electronics** | Rs. 18,000 | 52.94% | 52.94% | 2 (25.0%) |
| **Grocery** | Rs. 11,000 | 32.35% | 85.29% | 4 (50.0%) |
| **Clothing** | Rs. 5,000 | 14.71% | 100.00% | 2 (25.0%) |
| **Total** | **Rs. 34,000** | **100.00%** | **100.00%** | **8 (100.0%)** |

*Analytical Finding*: The dataset exhibits clear Pareto characteristics: while **Grocery** accounts for 50% of customer order volume, **Electronics** accounts for more than half (52.94%) of total revenue.

---

## 5. Acceptance Test Cases Verification Matrix

The application has been verified against all 4 official test cases from the specification with 100% output parity:

| Test Case | Scenario / Focus | Input Dataset | Expected Output Summary | Observed Status |
| :--- | :--- | :--- | :--- | :---: |
| **Test Case 1** | Normal Input (Monthly spending) | [`test_case_1.csv`](test_case_1.csv) | Jan: Rs. 5000, Feb: Rs. 12500, Mar: Rs. 5000, Apr: Rs. 11500, Total: Rs. 34000, Best Month: Feb | ✅ **PASS** |
| **Test Case 2** | Boundary / Single Order (Order stats) | [`test_case_2.csv`](test_case_2.csv) | Orders: 1, Mean: Rs. 2000.00, Median: Rs. 2000.00, Best Month: Jan | ✅ **PASS** |
| **Test Case 3** | Invalid Input (Validation warnings) | [`test_case_3.csv`](test_case_3.csv) | Warning on Row 9 (`N/A`) & Row 10 (`-500`); Valid orders analysed: 8 | ✅ **PASS** |
| **Test Case 4** | Category Share & Statistics | [`test_case_4.csv`](test_case_4.csv) | Electronics 52.94%, Grocery 32.35%, Clothing 14.71%, Mean: Rs. 4250.00, Median: Rs. 3000.00 | ✅ **PASS** |

---

### Terminal Execution Proofs (Individual Test Cases)

#### Test Case 1: Normal Input Verification (Monthly Spending)
![Test Case 1 Screenshot](test_case_1_screenshot.png)

#### Test Case 2: Boundary / Minimum Input Verification (Single Order)
![Test Case 2 Screenshot](test_case_2_screenshot.png)

#### Test Case 3: Invalid Input & Warnings Verification
![Test Case 3 Screenshot](test_case_3_screenshot.png)

#### Test Case 4: Category Share & Statistics Verification
![Test Case 4 Screenshot](test_case_4_screenshot.png)

---

## 6. Visualization Architecture (Matplotlib)

The visualization module in [`main.py`](main.py) produces an annotated bar chart (`monthly_spending.png`):
- **Resolution**: Rendered at $8 \times 5$ inches at 300 DPI for publication clarity.
- **Color Aesthetics**: Direct single-hue navy (`#2b5c8f`) with 0.55 bar width.
- **Value Annotations**: Every bar features its exact currency value centered above the column (`plt.annotate`).
- **Headless Safety**: Checks `os.environ.get("HEADLESS")` to prevent hanging in headless servers or automated testing environments.

---

## 7. Software Engineering & Defensive Standards

1. **Zero External Binaries**: Operates purely on standard Python 3 with verified ecosystem libraries (`pandas`, `numpy`, `matplotlib`).
2. **Defensive Data Handling**: Handles missing files, empty files, missing column headers, and non-numeric corrupted strings without unhandled exceptions.
3. **Chronological Sorting**: Ensures calendar sequence (`Jan` through `Dec`) through an explicit mapping index rather than lexicographical string sorting.
4. **Memory Hygiene**: Manages open Matplotlib figures with explicit `plt.close()` calls to eliminate GUI thread memory leaks.

---

## 8. Conclusion

The **Customer Purchase Pattern Analyzer** satisfies all functional and non-functional requirements. Its clean architecture, robust data sanitization engine, accurate NumPy vector computations, and verifiable test suite deliver a reliable, production-ready tool for retail and e-commerce transaction analysis.
