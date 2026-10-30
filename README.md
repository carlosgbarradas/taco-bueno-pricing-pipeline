# Taco Bueno Pricing & Promotion Analytics

## About the Project

We developed this project for the **Sun Holdings × Databricks Challenge** as third-semester Data Science and Mathematics Engineering students at Tecnológico de Monterrey.

Our goal is to explore Taco Bueno sales data to better understand products, prices, discounts, and customer purchases. These findings can help us propose improvements to promotions.

During the first stage, we worked on exploratory data analysis (EDA). For this stage, we connected our notebooks in a Databricks Job so the preparation, quality checks, and analyses can run in order without opening each notebook manually.

## Our Data

The dataset contains point-of-sale records from approximately 600 CSV files. Each row represents item-level activity within a customer order, so one ticket can have several rows.

Some of the main variables are:

- Store number, order date, and check number
- Menu item, quantity, and menu price
- Item discounts and ticket discounts
- Tax and total amounts
- Transaction type and cancellation flags

We use **store number + order date + check number** together to identify each ticket.

The supplied 2025 files were empty, containing only column headers. After processing the available files, the table contained **42,253,092 rows**, with business dates from **December 31, 2025, to September 27, 2026**.

The original CSV files are not included in this repository. Running the project requires access to the challenge dataset.

## Repository Files

| File | What it does |
|---|---|
| `01_data_preparation.ipynb` | Loads the CSV files, prepares the data, and saves the clean table |
| `02_data_quality_checks.ipynb` | Checks data quality and stops the pipeline if critical validations fail |
| `02_stores_and_times.ipynb` | Explores sales by store and time |
| `03_products_and_prices.ipynb` | Explores products and their prices |
| `04_discounts_and_baskets.ipynb` | Analyzes discounts and the items purchased together |
| `05_economia.ipynb` | Examines the business meaning of the data |
| `jobs/taco_bueno_pipeline.yml` | Stores the Job configuration and task dependencies |

## How the Pipeline Works

Our Databricks Job is called **`Taco_Bueno_Pipeline`**.

It follows this order:

1. **Prepare the data.**
2. **Run the quality checks.**
3. **Run the four analysis notebooks in parallel**, once the quality task succeeds.

All analysis tasks depend on the quality task and use the **All succeeded** condition.

The Job uses serverless compute. We started the execution manually.

## Data Preparation and Quality Checks

The preparation notebook creates the Delta table:

`workspace.default.taco_bueno_clean`

All analysis notebooks use this shared table.

We check required columns, selected data types, missing ticket identifiers, price calculations, discount calculations, and cancellation flags.

We also added a final validation step that raises an error if it finds a critical problem, such as:

- An empty table
- Missing ticket identifiers
- Incorrect data types in the columns being checked
- Incorrect net amount calculations or reconciliation statuses
- Different net amounts within the same reconciled ticket
- Incorrect discount percentages or cancellation flags

If this step fails, the dependent analyses do not run.

Some source records still require review. These are identified with status fields instead of being treated as valid sales. Correctly identified exceptions do not automatically stop the pipeline.

## Important Rules for Using the Table

One important part of this project was distinguishing item-level amounts from ticket-level amounts.

- `net_item_price` represents the net amount for an item line: quantity × menu price − item discount.
- `net_total` represents a ticket-level net amount before tax.
- Since a ticket can appear on several rows, `net_total` must be counted only once per ticket.
- We use `net_total` for revenue analysis only when `net_total_status` is `reconciled`.
- Store exclusion flags and other filters must be considered when interpreting results.
- The same product appearing more than once in a ticket does not necessarily mean that the data is duplicated.

In the reviewed quality report, **5,948,724 tickets** had validated net amounts within the included-store scope. This represented approximately **98.94%** of tickets in that scope.

## How to Run the Project

1. Get access to the original challenge CSV files.
2. Import the six notebooks into Databricks.
3. Update the input and output paths to match your Workspace.
4. Make sure your account has permission to read the files, create or update the clean table, and save outputs.
5. Create a Job using the tasks and dependencies described above.
6. Use `jobs/taco_bueno_pipeline.yml` as a reference for the configuration.
7. Set dependent tasks to **All succeeded**.
8. Run the Job and review the quality report and analysis results.

The YAML file contains paths from our Workspace. These need to be changed when using another Workspace. It is an exported Job configuration, not a complete deployment package.

## Successful Execution

We ran the complete pipeline on **October 29, 2026**, after adding the final quality validation.

| Item | Result |
|---|---|
| Job | `Taco_Bueno_Pipeline` |
| Status | Succeeded |
| Completed tasks | 6 of 6 |
| Total duration | 7 minutes 39 seconds |
| Run ID | `813940331475255` |

This run shows that all six notebooks completed and the configured quality checks passed.

## Results and Limitations

The notebooks generate tables and charts about stores, sales timing, products, prices, discounts, and shopping baskets. Output locations are defined in each notebook.

Some analyses use specific date ranges or a limited number of stores or products to make comparisons easier to read. These selections should be considered before applying a finding to the entire dataset.

Our analysis is exploratory. For example, finding that tickets with discounts have higher average spending does not prove that the discount caused the difference. Product combinations, store differences, and purchase timing may also affect the results.

We also cannot conclude that a promotion is more profitable based only on sales, because profitability requires cost information.

## What We Learned

This project helped us practice working with a large dataset, checking calculations, and organizing our analysis into a repeatable process.

We learned that getting a notebook to run is only one part of the work. We also need to understand what each row represents, avoid counting ticket totals multiple times, and explain the limitations of our results.

Connecting the notebooks in a Job helped us bring the team's work together and made the execution process easier to follow.
