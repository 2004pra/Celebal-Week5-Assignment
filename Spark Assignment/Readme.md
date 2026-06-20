# Spark Assignment - Week 5

## Objective
Understand Spark fundamentals and use PySpark DataFrames to clean, transform, filter, and aggregate data.

## Dataset
[Sample Superstore dataset](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) - retail order data with ~10,000 rows covering Sales, Profit, Category, Region, Customer info, and Order/Ship dates.

## What's in this repo

```
spark-assignment/
│── data/
│   └── dataset.csv          # raw Superstore data
│── notebook/
│   └── spark_basics.ipynb   # all the code, step by step
│── output/
│   ├── Result
.csv     # full dataset after cleaning + filtering
│   └── category_summary.csv # groupBy aggregation by Category
│── README.md
```

## Why Spark over MapReduce
MapReduce reads/writes to disk between every step, so anything with multiple stages gets slow fast. Spark keeps data in memory across operations instead, which is the main reason it's faster for iterative or multi-step pipelines like this one. DataFrames on top of that give you a much easier, almost SQL-like way to work with structured data instead of writing raw map/reduce functions by hand.

## What the notebook does

1. **Setup** - installs PySpark (needed on Colab since it's not preinstalled) and starts a SparkSession.
2. **Load data** - reads the CSV into a DataFrame, checks schema, row count, and a sample of rows.
3. **Cleaning**
   - Dropped duplicate rows with `dropDuplicates()`
   - Checked nulls per column
   - Filled missing `Postal Code` values, dropped rows with null `Sales`
4. **Filtering** - filtered by `Category`, `Region`, and a `Sales` threshold to practice condition-based filtering.
5. **Transformation** - renamed columns (`Sales` → `Total_Sales`) and explicitly cast `Sales` to `DoubleType` instead of trusting Spark's schema inference, which actually matters here (see Issues section below).
6. **Aggregation** - count, sum, avg, min, max on `Sales` and `Profit`.
7. **GroupBy** - grouped by `Category` and `Region`, then filtered on the aggregated totals (e.g. categories with total sales above a threshold).
8. **Wide transformations / shuffle** - `groupBy` and `orderBy` both trigger a shuffle since data has to move across partitions to group/sort correctly. Just noting the concept here, not going deep into partition tuning.
9. **Pipeline** - chained all of the above (read → clean → filter → transform → aggregate) into one pipeline, output saved as CSV.

## Issues hit along the way (and fixes)

Worth documenting since these were actual bugs, not made up:

- **Schema inference going wrong**: `inferSchema=True` mis-typed the `Sales` column (sometimes as integer, sometimes confused entirely) because some `Product Name` values contain embedded commas and quotes, which threw off the default CSV parser and shifted columns for some rows. Fixed by reading with `quote='"'`, `escape='"'`, `multiLine=True`, and additionally force-casting `Sales` to `DoubleType` after load rather than relying on inference alone.
- **Spark doesn't write a single CSV file** - `.write.csv()` writes a folder of partitioned `part-*.csv` files, not one clean file. Used `.coalesce(1)` before writing, then renamed the single part file to the actual output filename.

## Observations from the output
- Technology and Office Supplies categories carry the bulk of total sales volume.
- A handful of orders have negative profit despite positive sales, worth a closer look if this were a real analysis (likely high-discount orders).
- After cleaning there were very few actual duplicate/null rows in this dataset, so most of the cleaning step is here to show the process rather than fix major data quality problems.

## How to run
1. Open `notebook/spark_basics.ipynb` in Google Colab.
2. Run the first cell to install PySpark.
3. Upload `data/dataset.csv` (or mount Drive) and update the file path if needed.
4. Run all cells top to bottom.
