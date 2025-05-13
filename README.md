# 📷 Camera Order Data Analysis

## 📊 Dataset Description

This dataset simulates camera purchase behavior on an e-commerce platform. It includes 2,000 synthetic records of customer orders with the following key attributes:

* **Timestamp**: Date and time of order placement
* **Customer and Associate Details**: Unique IDs and names
* **Order Information**: Product category (camera-related items), workflow type, issue resolution times
* **Performance Metrics**: SLA compliance, first response times, reopen count, and escalation flags
* **Team Data**: Manager, shift, site location, and ticketing tool used

The dataset is designed to replicate real-world customer support and operations scenarios for analytical use.

## 🎯 Purpose of the Analysis

The goal of this analysis is to:

* Evaluate associate and team performance using operational KPIs
* Identify trends in order resolution, escalations, and SLA breaches
* Explore relationships between product categories and support workflows
* Generate insights for process improvement and strategic decision-making
* Practice data cleaning, transformation, and visualization techniques using Python and relevant data libraries

## Analysis

### 1. Combining Tables to Make One Table

* **Purpose:** This section merges the three individual DataFrames (`customers_df`, `asins_df`, and `orders_df`) into a single, comprehensive DataFrame (`final_output_df`). This combined table contains all the necessary information from the three original sources linked together by common identifiers (`customer_id` and `asin`).
* **Code Steps:**
    1.  Import the `pandas` library.
    2.  Merge `orders_df` with `asins_df` using an inner join on the 'asin' column. This brings product details (item name, brand, category) into the orders data.
    3.  Merge the result of the first join with `customers_df` using an inner join on the 'customer\_id' column. This adds customer details (customer name) to the combined order and product data.
    4.  Select and reorder the desired columns (`customer_id`, `customer_name`, `order_id`, `order_date`, `shipping_channel`, `order_status`, `asin`, `item_name`, `brand`, `category`) to form the final DataFrame `final_output_df`.
    5.  Print the head of the `final_output_df` and its shape.
    6.  Save the `final_output_df` to a CSV file named `combined_order_details.csv`.
* **Findings (from PDF output):**
    * The resulting `final_output_df` has 100,000 rows and 10 columns, successfully combining data from all three original tables.
    * The head of the DataFrame shows the structure with the selected columns.
    * The data was successfully saved to a CSV file.

### 2. EDA (Exploratory Data Analysis)

* **Purpose:** This section performs initial analysis to understand the characteristics, quality, and patterns within the combined dataset.
* **Code Steps:**
    1.  Import necessary libraries: `pandas`, `numpy`, `matplotlib.pyplot`, `seaborn`.
    2.  Set the plot style using `seaborn`.
    3.  **Data Cleaning & Inspection:**
        * Print the shape of the `final_output_df`.
        * Use `.describe()` to get descriptive statistics for numerical columns.
        * Use `.info()` to check data types and non-null counts for all columns.
        * Use `.isnull().sum()` to count missing values per column.
        * Use `.nunique()` to count unique values per column.
    4.  **Order Status and Fulfillment Analysis:**
        * Plot the distribution of `order_status` using a count plot.
        * Plot the distribution of `shipping_channel` usage (excluding nulls) using a count plot.
        * Plot the distribution of `order_status` broken down by `shipping_channel` using a count plot with `hue`.
        * (Attempted) Shipping Time Analysis: The code in the PDF attempts to calculate shipping duration by converting `order_date` and `shipping_date` to datetime, calculating the difference, and visualizing the distribution and variation by `shipping_channel` and `is_fba`. *Note: Based on the combined data head shown in the PDF, `shipping_date` and `is_fba` were not included in `final_output_df`. The code includes checks and potential merges to handle this, but the direct calculation as written might not work without merging these columns back from the original `orders_df`.*
    5.  **Customer Activity Analysis:**
        * Calculate and print the distribution of orders per customer name (`value_counts()`).
        * Identify and print the top 10 customers by order count.
        * (Attempted) Identify top customers by total amount spent: The code attempts to merge `total_amount` from the original `orders_df` and then group by customer name to sum the total amount. *Note: Similar to shipping date, `total_amount` was not in the combined data head, requiring this merge.*
        * (Attempted) Customer Origin Analysis: The code attempts to merge `country_of_origin` from the original `customers_df` and plot the distribution. *Note: `country_of_origin` was also not in the combined data head, requiring this merge.*
    6.  **Product, Brand, and Category Performance Analysis:**
        * Identify and print the top 10 most popular ASINs by order count and quantity sold.
        * Identify and print the most popular brands by order count, quantity sold, and total amount spent (requires `total_amount` merge).
        * Identify and print the most popular categories by order count, quantity sold, and total amount spent (requires `total_amount` merge).
        * Verify Brand-Category relationship by creating a pivot table showing counts of items by brand and category.
        * Analyze performance (e.g., Total Amount) by Brand and Category (requires `total_amount` merge).
* **Findings (from PDF output):**
    * Shape: (100000, 10).
    * Data Types: `customer_id` (int64), `order_date` (datetime64[ns]), others (object). `shipping_channel` has nulls.
    * Missing Values: 14909 missing values in `shipping_channel`, corresponding to Cancelled/Lost orders.
    * Unique Values: 500 unique customers, 100000 unique orders, 1000 unique ASINs/item names, 7 brands, 5 categories.
    * Order Status: Around 85% Delivered, 10% Cancelled, 5% Lost in transit.
    * Shipping Channel: Amazon is the most used channel among shipped orders.
    * Order Status by Channel: Primarily Delivered for all channels shown.
    * Customer Activity: Shows distribution of orders per customer (many customers with similar high order counts), top customers by order count, and top customers by total amount (if data available).
    * Product/Brand/Category Performance: Identifies top ASINs, brands (Sony, Fujifilm, Amazon Basics are most popular by order count), and categories (Camera, Camera Lens most popular by total amount). The Brand-Category matrix confirms the relationships.

**4. Prepare for Co-occurrence Analysis**

* **Purpose:** This section prepares data and performs analysis to identify pairs of ASINs that are frequently ordered together based on two specific criteria.
* **Code Steps:**
    1.  Import necessary libraries: `pandas`, `numpy`, `matplotlib.pyplot`, `seaborn`, `collections.Counter`, `itertools.combinations`.
    2.  Set plot style.
    3.  Filter for 'Delivered' orders.
    4.  Ensure `order_date` is datetime type.
    5.  **Scenario i: ASINs Ordered by Same Customer from Same Brand (Different Occasions):**
        * Group delivered orders by `customer_id` and `brand`.
        * Collect the unique `asin`s for each group into a list.
        * Filter groups with more than one unique ASIN.
        * Iterate through these groups, generate unique pairs of ASINs using `itertools.combinations`, and count how many *customers* ordered each pair from that brand using `collections.Counter`.
        * Convert the counts into a DataFrame `frequently_ordered_diff_occasions_df`.
        * Print the head of this DataFrame.
    6.  **Scenario ii: ASINs Ordered Together (Same Customer, Same Brand, Same Order Date):**
        * Group delivered orders by `customer_id`, `order_date`, and `brand`.
        * Collect the `asin`s for each group into a list.
        * Filter groups with more than one ASIN.
        * Iterate through these groups, generate unique pairs of ASINs using `itertools.combinations`, and count how many *times* each pair was ordered together in such groups using `collections.Counter`.
        * Convert the counts into a DataFrame `frequently_ordered_same_day_df`.
        * Print the top 10 rows of this DataFrame.
        * Plot the top 10 pairs using a bar plot.
* **Findings (from PDF output):**
    * Two DataFrames are created: `frequently_ordered_diff_occasions_df` and `frequently_ordered_same_day_df`.
    * `frequently_ordered_diff_occasions_df` shows pairs of ASINs from the same brand ordered by the same customer over time, with customer counts.
    * `frequently_ordered_same_day_df` shows pairs of ASINs from the same brand ordered in the same transaction, with order counts. The top 10 pairs and their counts are shown in a table and a bar plot.

**5. Calculating Brand Repeat Purchase Rate**

* **Purpose:** To determine the percentage of customers who placed more than one order for a specific brand.
* **Code Steps:**
    1.  Filter for delivered orders.
    2.  Group by `customer_id` and `brand` and count unique orders per group.
    3.  Identify repeat customers for each brand (where order count > 1).
    4.  Count the number of repeat customers per brand.
    5.  Count the total number of unique customers per brand.
    6.  Merge the repeat and total customer counts.
    7.  Calculate the repeat purchase rate (`repeat_customer_count / total_customer_count * 100`).
    8.  Print the repeat purchase rate by brand, sorted descending.
    9.  Visualize the repeat purchase rates using a bar plot.
* **Findings (from PDF output):**
    * All brands except Nike and Hasselblad have a 100% repeat purchase rate among customers who ordered them at least once. Nike has 99.8% and Hasselblad has 99.0%. This suggests the synthetic data heavily favored repeat purchases for brands once a customer bought from them.

**6. Analyzing First Brand and Repeat Purchase Behavior**

* **Purpose:** To analyze which brands customers buy first and whether repeat customers stick to their initial brand or explore others.
* **Code Steps:**
    1.  Filter for delivered orders and sort by `customer_id` and `order_date`.
    2.  Identify the first order for each customer and extract the `first_brand` and `first_category`.
    3.  Identify repeat customers (more than 1 delivered order).
    4.  Filter orders to include only repeat customers.
    5.  Merge the `first_brand` and `first_category` information into the repeat customer orders.
    6.  Assign an order sequence number to identify subsequent orders.
    7.  Filter for subsequent orders (`order_sequence > 1`).
    8.  Group subsequent orders by customer and collect the set of `subsequent_brands` and `subsequent_categories`.
    9.  Merge this summary with the first brand/category information.
    10. Categorize repeat customers' brand behavior (e.g., 'Only Bought First Brand Again', 'Bought First Brand Again and Other Brands', 'Bought Only Other Brands').
    11. For customers who bought other brands, categorize their category behavior (e.g., 'Stuck to First Category', 'Bought First Category and Other Categories', 'Switched to Other Categories').
    12. Summarize and visualize the counts/percentages for these behavior categories using bar plots.
    13. (Optional) Analyze brand switching pairs (first brand -> subsequent brand).
* **Findings (from PDF output):**
    * Shows the total number of unique customers with delivered orders and the number of repeat customers.
    * Provides counts/percentages for brand behavior categories, illustrating how many repeat customers stuck to their first brand, explored others, or did both.
    * Provides counts/percentages for category behavior categories for customers who bought other brands, showing whether they stuck to the initial category or explored others.
    * (Optional) Shows the top brand switching pairs if any were observed.



 ### 4. Code Documentation

 * **`pd.DataFrame()`**
    * **Definition:** Creates a DataFrame object.
    * **Syntax:** `pd.DataFrame(data, index, columns, dtype, copy)`
    * **Context in Notebook:** Used in the data generation section (which we are excluding from detailed description but acknowledging its use here) to create the initial `customers_df`, `asins_df`, and `orders_df`. It's also used to create new DataFrames from the results of aggregations or calculations, such as `frequently_ordered_same_day_df` and `frequently_ordered_diff_occasions_df` from `Counter` outputs.

* **`df.merge()`**
    * **Definition:** Merges DataFrame or named Series objects with a database-style join.
    * **Syntax:** `df1.merge(df2, on=None, how='inner', ...)`
    * **Context in Notebook:** Used in the "Combining tables" section to join `orders_df` with `asins_df` and then the result with `customers_df` based on common columns ('asin' and 'customer\_id') to create `final_output_df`. Also used in EDA to merge back `shipping_date`, `is_fba`, `total_amount`, and `country_of_origin` from original DataFrames for specific analyses.

* **`df[...]` (Column Selection/Indexing)**
    * **Definition:** Selects one or more columns from a DataFrame.
    * **Syntax:** `df['column_name']` (for a single column, returns a Series), `df[['col1', 'col2']]` (for multiple columns, returns a DataFrame).
    * **Context in Notebook:** Used extensively to select specific columns like `asin`, `item_name`, `brand`, `category`, `customer_id`, `customer_name` during the merge operations and throughout the EDA and co-occurrence analysis to access data in specific columns.

* **`df.head()`**
    * **Definition:** Returns the first `n` rows of the DataFrame.
    * **Syntax:** `df.head(n=5)`
    * **Context in Notebook:** Used after creating or modifying DataFrames to display the initial rows and verify the structure and content, such as printing the head of `final_output_df` and the co-occurrence results.

* **`len(df)`**
    * **Definition:** Returns the number of rows in the DataFrame. (Note: `len()` is a built-in Python function, but commonly used with pandas DataFrames).
    * **Syntax:** `len(df)`
    * **Context in Notebook:** Used to print the total number of rows in the `final_output_df` after combining tables.

* **`df.to_csv()`**
    * **Definition:** Writes the DataFrame to a comma-separated values (CSV) file.
    * **Syntax:** `df.to_csv(path_or_buf, index=True, ...)`
    * **Context in Notebook:** Used in the "Combining tables" section to save the `final_output_df` to `combined_order_details.csv`. `index=False` is used to prevent writing the DataFrame index as a column.

* **`df.shape`**
    * **Definition:** Returns a tuple representing the dimensions of the DataFrame (rows, columns).
    * **Syntax:** `df.shape`
    * **Context in Notebook:** Used in the EDA section to check and print the shape of the `final_output_df`.

* **`df.describe()`**
    * **Definition:** Generates descriptive statistics for numerical columns in the DataFrame.
    * **Syntax:** `df.describe(percentiles=None, include=None, exclude=None, ...)`
    * **Context in Notebook:** Used in the EDA section to display summary statistics (count, mean, std, min, max, quartiles) for the numerical columns in `final_output_df`.

* **`df.info()`**
    * **Definition:** Prints a concise summary of a DataFrame, including the index dtype and column dtypes, non-null values and memory usage.
    * **Syntax:** `df.info()`
    * **Context in Notebook:** Used in the EDA section to check the data types of each column and the number of non-null entries.

* **`df.isnull()`**
    * **Definition:** Detect missing values in the DataFrame. Returns a boolean DataFrame of the same shape.
    * **Syntax:** `df.isnull()`
    * **Context in Notebook:** Used in the EDA section in conjunction with `.sum()` to count the number of missing values per column.

* **`df.isnull().sum()`**
    * **Definition:** Calculates the sum of boolean values (True is 1, False is 0) along each column, effectively counting the number of `True` values (missing values) in each column.
    * **Syntax:** `df.isnull().sum()`
    * **Context in Notebook:** Used in the EDA section to get a count of missing values for each column in `final_output_df`.

* **`df.nunique()`**
    * **Definition:** Counts the number of distinct non-null values in each column.
    * **Syntax:** `df.nunique(axis=0, dropna=True)`
    * **Context in Notebook:** Used in the EDA section to determine the number of unique entries in each column, providing insight into the cardinality of features like `customer_id`, `order_id`, `brand`, etc.

* **`df['column'].value_counts()`**
    * **Definition:** Returns a Series containing counts of unique values in a Series in descending order.
    * **Syntax:** `series.value_counts(normalize=False, sort=True, ascending=False, ...)`
    * **Context in Notebook:** Used extensively in EDA to analyze the distribution of categorical data, such as counting orders per `order_status`, `shipping_channel`, `customer_name`, `brand`, and `category`. Also used to get the top N most frequent items/brands/categories.

* **`df.dropna()`**
    * **Definition:** Remove missing values.
    * **Syntax:** `df.dropna(axis=0, how='any', subset=None, ...)`
    * **Context in Notebook:** Used in the EDA section to filter out rows where `shipping_channel` is missing before analyzing shipping channel usage and order status by channel.

* **`df[condition]` (Boolean Indexing)**
    * **Definition:** Selects rows from a DataFrame where the boolean condition is True.
    * **Syntax:** `df[df['column'] > value]` or `df[df['column'].isin(list)]`
    * **Context in Notebook:** Used frequently to filter the DataFrame, such as selecting only 'Delivered' orders for analysis, filtering for repeat customers, or selecting rows with missing `shipping_channel`.

* **`pd.to_datetime()`**
    * **Definition:** Converts argument to datetime objects.
    * **Syntax:** `pd.to_datetime(arg, errors='coerce', ...)`
    * **Context in Notebook:** Used in the EDA section to convert the 'order\_date' and 'shipping\_date' columns to datetime objects, which is necessary for time-based analysis and calculations like shipping duration.

* **`df['date_col'].dt.days`**
    * **Definition:** Accesses the datetime properties of a Series of datetime objects and returns the number of days.
    * **Syntax:** `series.dt.days`
    * **Context in Notebook:** Used in the EDA section to calculate the shipping duration in days by subtracting the `order_date` from the `shipping_date` (which results in a Timedelta Series) and then accessing the `.dt.days` attribute.

* **`df.groupby()`**
    * **Definition:** Groups DataFrame using a mapper or by a Series of columns.
    * **Syntax:** `df.groupby(by=None, axis=0, ...)`
    * **Context in Notebook:** Used extensively for aggregation and co-occurrence analysis. It groups data by columns like `customer_id`, `brand`, `order_date`, or combinations thereof to perform calculations or collect related data within those groups.

* **`groupby().agg()`**
    * **Definition:** Performs aggregation(s) on the grouped data.
    * **Syntax:** `groupby_object.agg(func, *args, **kwargs)` or `groupby_object.agg({'col1': 'agg_func1', 'col2': 'agg_func2'})`
    * **Context in Notebook:** Used after `groupby()` to apply aggregation functions like `'nunique'` (to count unique order IDs or customer IDs), `'sum'` (to sum quantities or total amounts), and `list` or `set` (to collect ASINs within groups).

* **`groupby().size()`**
    * **Definition:** Compute group sizes. Returns a Series with group names as index.
    * **Syntax:** `groupby_object.size()`
    * **Context in Notebook:** Used to count the total number of delivered orders for each customer.

* **`groupby().head(n)`**
    * **Definition:** Select the first `n` rows of each group.
    * **Syntax:** `groupby_object.head(n=5)`
    * **Context in Notebook:** Used in the analysis of first brand purchases to get the first order row for each customer after sorting by date.

* **`df.sort_values()`**
    * **Definition:** Sorts DataFrame by column or index values.
    * **Syntax:** `df.sort_values(by, axis=0, ascending=True, ...)`
    * **Context in Notebook:** Used to sort orders by customer and date to correctly identify the first order and to sort results (e.g., top customers by total amount, brands by repeat purchase rate).

* **`df.reset_index()`**
    * **Definition:** Reset the index, or a level of it.
    * **Syntax:** `df.reset_index(level=None, drop=False, name=None, inplace=False, ...)`
    * **Context in Notebook:** Used after `groupby().agg()` or `groupby().size()` to convert the grouped columns (which become the index of the result) back into regular columns in a new DataFrame.

* **`df['column'].apply()`**
    * **Definition:** Invoke function on values of Series or DataFrame.
    * **Syntax:** `series.apply(func, convert_dtype=True, args=())` or `df.apply(func, axis=0, ...)`
    * **Context in Notebook:** Used in the co-occurrence analysis to apply a lambda function to the list of ASINs within each group (e.g., to get the length of the list or convert to a set).

* **`df.copy()`**
    * **Definition:** Make a copy of this object’s indices and data.
    * **Syntax:** `df.copy(deep=True)`
    * **Context in Notebook:** Used after filtering a DataFrame (e.g., `delivered_orders_df = final_output_df[...]`) to ensure modifications to the new DataFrame do not affect the original one and to avoid the `SettingWithCopyWarning`.

* **`df['column'].astype()`**
    * **Definition:** Cast a pandas object to a specified dtype `dtype`.
    * **Syntax:** `series.astype(dtype, copy=True, errors='raise')`
    * **Context in Notebook:** Used in the co-occurrence analysis to convert the tuple representation of ASIN pairs into a string format (`.astype(str)`) for use as labels in plots. Also used to ensure integer types (`.astype(int)`).

* **`df['column'].fillna()`**
    * **Definition:** Fill NA/NaN values using the specified method.
    * **Syntax:** `series.fillna(value=None, method=None, ...)`
    * **Context in Notebook:** Used when calculating repeat purchase rate to fill `NaN` values in the 'repeat\_customer\_count' column (which occur for brands with no repeat customers) with `0`.

* **`df['column'].notna()`**
    * **Definition:** Detect existing (non-missing) values. Returns a boolean Series.
    * **Syntax:** `series.notna()`
    * **Context in Notebook:** Used in the shipping time analysis to filter for rows where the `shipping_date` is not missing.

* **`df['column'].tolist()`**
    * **Definition:** Convert the Series to a list.
    * **Syntax:** `series.tolist()`
    * **Context in Notebook:** Used to get a list of customer IDs for repeat customers.

* **`df.unstack()`**
    * **Definition:** Pivot a level of the (preferably hierarchical) index labels.
    * **Syntax:** `series.unstack(level=-1, fill_value=None)`
    * **Context in Notebook:** Used after grouping by two columns (e.g., `brand` and `category`) and using `.size()` to reshape the resulting Series into a DataFrame where one level of the index becomes columns.

* **`df.isin()`**
    * **Definition:** Whether elements in DataFrame are contained in `values`.
    * **Syntax:** `df.isin(values)`
    * **Context in Notebook:** Used to filter the DataFrame to include only rows where the `customer_id` is present in the list of `repeat_customers_ids`.

* **`groupby().cumcount()`**
    * **Definition:** Number of items in each group up to the current row.
    * **Syntax:** `groupby_object.cumcount(ascending=True)`
    * **Context in Notebook:** Used to assign a sequential number to each order within a customer's orders, allowing identification of the first order (`== 0` or `== 1` depending on start) and subsequent orders (`> 0` or `> 1`).
