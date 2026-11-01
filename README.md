## Day 2 - Sales Analytics Without Pandas
Day 2 - Sales Analytics Without Pandas

September 2026 sales report for Tech Hub Kenya (50 till rows, 5 stores). Tasks 1-7 use only the Python standard library, so the script runs on the reporting server as-is. pandas is used only in task 8, to check the answers on a laptop.

### How to run

```python
day2-sales-analytics/
├── data/raw/electronics_sales.csv
├── notebooks/sales_analytics.ipynb
├── output/sales_report.txt
├── requirements.txt
└── README.md
```
1. `uv sync` (pandas and a Jupyter kernel must be installed in the project environment, e.g. `uv add pandas ipykernel`).
2. Open `notebooks/sales_analytics.ipynb` and select the project's `.venv` kernel.
3. Run all cells in order.

The data path is `../data/raw/electronics_sales.csv`, which is relative to the notebook's own folder.

### Cleaning Rules and Why
A value is repaired only when it has one sensible meaning. Otherwise the row is skipped and listed with its sale id and reason, so finance can correct it at the source.

| **Situation**               | **Decision**          | **Reason**                    |
| :---:                       | :---:                 | :---:                         |
| Spaces around a value       | Repair(strip)         | Cannot change the meaning     |
| `KES 18999` (currency label,<br>any letter case)| Repair (remove KES)| The column is<br> in KES, so the label adds nothing|
| `38,999` (comma separator)  | Repair (remove comma) | Commas appear as thousand <br>separators|
| Number word in a word field | Repair(convers to the <br>number)| `two` in a quantity<br> means 2 and dropping it would<br> throw a real sale|
| Blank, `N/A`,`NONE`,`Null`,<br>`-`| Skip            | There is no number to recover  |


If either the quantity or the unit price is unusable, the whole row is skipped, because revenue needs both. Skipped rows are never filled in from other rows. For example, S-3030 is an HP 15 Laptop, which sells for 64,999 elsewhere in the file, but a discount or promotion could mean this sale was different, and the report would then show a number nobody recorded.

Revenue for each row is quantity x unit price.

### Ties in the top 3
Products are ranked with `sorted(..., key=lambda item:(-item[1], item[0]))`: highest revenue first, and equal revenue is ordered alphabetically by product name. There is no tie in this data, but the rule keeps the order stable if one appears.

### Results
| Rows read/used/skipped     |              50/47/3|
| Total revenue              |     KES 1,317,210.00|
| Mean transaction revenue   |            28,025.74|
| Median transaction revenue |            10,998.00|
| Min/max transaction revenue|1,499.00 / 259,996.00|

#### Top 3 products
1. HP 15 Laptop - KES 389,994.00
2. Hisense 43in Smart TV - KES 272,993.00
3. Lenovo Tab M10 - KES 199,992.00

**Skipped rows**: S-3030 (unit price `N/A`), S-3035(quantity blank), S-3039(unit price blank).

**High-value transcations (at or above KES 50,000)**: 5 - S-3013, S-3017, S-3022, S-3036 and S-3048. By store, Kisumu Mega Plaza has 2, Nairobi CBD, Nakuru Westside and Westgate have 1 each, and Mombasa City Mall has 0. S-3003 (Lenovo Tab M10, 2 x 24,999 = 49,998) falls just below the threshold, so it is correctly not included.

The mean is well above the median because a few laptop and TV orders are very large.

### Pandas Verification
`verify_with_pandas` reads the CSV with pandas, applies the notebooks `parse_number` to the quantity and price columns, drops the unusable rows, and recomputes the total, the revenue by product, the top 3 and the mean and median. Result:

```python
Total revenue           : MATCH(pure=1317210.00, pandas=1317210.00)
Revenue by product      : MATCH
Top 3                   : MATCH
Mean / median           : MATCH 
```
**What this check covers:** it confirms that the summing, grouping and ranking logic gives the same answers when done with pandas. Because it reuses `parse_number`, it does not independently test the cleaning rules, and the mean and median in the check are calculated from the pandas-cleaned revenue values with plain Python rather than with pandas' own `mean()` and `median()`.

#### Known limits

These do not occur in the September file, but they would matter if the till export changes:

- All commas are removed, so a decimal comma such as 1,5 would be read as 15.
- Negative numbers, zero, exponent notation such as 1e3, and inf are accepted by float(), and there is no check that a price is above zero or a quantity is at least 1.
- A fractional quantity such as 2.5 is stored as 2 by int(), while the revenue is still calculated from 2.5.
- There is no check for duplicate sale_id values or for dates outside September 2026.
