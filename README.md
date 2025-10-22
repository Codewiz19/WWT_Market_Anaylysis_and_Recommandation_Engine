# WWT Market Analysis and food Recommandation problem 

# WWT Unravel 2025: "Wings R Us" Recommendation Engine

[cite_start]This repository contains the complete solution for the WWT Unravel 2025 competition[cite: 1, 2]. [cite_start]The project focuses on developing a recommendation algorithm for the quick-service restaurant (QSR) client, "Wings R Us," to enhance their digital checkout experience[cite: 6, 8, 23].

## 1. The Business Problem

[cite_start]**Client:** "Wings R Us," a US-based QSR chain specializing in chicken wings[cite: 6].

[cite_start]**Business Goal:** The client wants to enhance their digital checkout experience on their mobile app, website, and kiosks by introducing **smart, last-minute item recommendations**[cite: 23, 30].

**Key Objectives:**
* [cite_start]**Increase Average Order Value (AOV)**[cite: 23].
* [cite_start]Enhance user satisfaction[cite: 8].
* [cite_start]Improve customer retention[cite: 8].
* [cite_start]Increase basket size[cite: 8].

## 2. The Technical Challenge

[cite_start]The core task is to build a recommendation engine that suggests relevant items to a user based on the items already in their cart[cite: 8, 23].

* [cite_start]**Deliverable:** A recommendation model that, given a customer's cart with one item removed, predicts the top 3 most likely missing items[cite: 63, 124].
* [cite_start]**Evaluation Metric:** The solution is scored based on **Recall@3**[cite: 126, 127]. [cite_start]This metric measures how often the *actual* missing item is present within our model's top 3 recommendations[cite: 125, 126].

## 3. My Approach: Market Basket Analysis

This problem is a classic example of **Market Basket Analysis**. The goal is to find items that are "frequently bought together."

My solution uses **Association Rule Mining** to discover these patterns. This algorithm generates "if-then" rules from historical order data. For example:

> **IF** a user buys `{6pc Boneless Mild}` and `{Large Cheese Fries}`...
> **THEN** they are 80% likely to also buy a `{20 Oz Soda}`.

This approach is powerful because it's:
* **Interpretable:** The business can understand *why* an item is being recommended.
* **Direct:** It directly uses the co-occurrence of items in the cart, which is the strongest signal available in the provided data.
* **Effective:** It's a time-tested and standard algorithm for "frequently bought together" recommendations.

## 4. Data Processing & Exploratory Data Analysis (EDA)

[cite_start]This project uses four primary datasets: `order_data` [cite: 43][cite_start], `customer_data` [cite: 49][cite_start], `store_data` [cite: 51][cite_start], and the `test_data_question` set[cite: 45]. The full process is detailed in `EDA.ipynb`.

[cite_start]The most critical step was processing the `ORDERS` column in the `order_data` file[cite: 41]. This column was stored as a JSON string, not a usable format.

**Data Transformation Pipeline:**
1.  **Load & Merge:** All `*.csv` files were loaded into pandas DataFrames and merged into a single master dataset.
2.  **Parse JSON String:** The `ORDERS` column, which looked like `'[{...}, {...}]'`, was safely parsed using `ast.literal_eval`.
3.  **Explode Rows:** The dataset was "exploded" so that each item in a single order (e.g., wings, fries, soda) was transformed from a list into its own individual row.
4.  **Normalize JSON:** The nested JSON within each item was normalized to extract key features like `item_name` and `quantity`.

**Key EDA Finding:**
A frequency analysis of all items was performed to identify the **Top 20 Most Popular Items**. This list (e.g., "Large Seasoned Fries," "20 Oz Soda") became the foundation for our "cold start" fallback strategy.

## 5. The Recommendation Engine (`WWT_REC.ipynb`)

The core recommendation logic is built using the `mlxtend` library.

### Step 1: Create Transactional Data
The cleaned, exploded data was grouped by `ORDER_ID`. This created a "transaction" for each order, which is simply a list of all unique items purchased in that order.

### Step 2: One-Hot Encoding
To prepare the data for the Apriori algorithm, these transactions were converted into a one-hot encoded DataFrame.
* Each **row** represents a unique `ORDER_ID`.
* Each **column** represents a unique `item_name`.
* The value is `1` if the item was in the order, and `0` if it was not.

### Step 3: Train the Model & Generate Rules
The `apriori` algorithm was used to find "frequent itemsets" (groups of items that appear together often). From these itemsets, `association_rules` were generated.

These rules were then filtered by `confidence` (the "THEN" likelihood) to ensure all recommendations are statistically strong.

### Step 4: Generate Recommendations
For each order in the `test_data_question` file, the following logic is applied:
1.  [cite_start]The items already in the user's cart are identified (e.g., `{Hot Honey Rub (Boneless), Garlic Parmesan (Tenders)}`)[cite: 96, 97, 98].
2.  The master list of association rules is filtered to find all rules where the `antecedents` (the "IF" part) are a subset of the items in the cart.
3.  These matching rules are sorted in descending order by `confidence` and `lift` to find the *strongest* and *most relevant* recommendations.
4.  The `consequents` (the "THEN" part) of the top 3 rules are extracted. These are our recommendations.

## 6. The "Cold Start" Problem & Fallback Strategy

**The Problem:** What happens if a user's cart is unique and doesn't match *any* rules? This is known as the "cold start" problem. The model would fail to return any recommendations.

**The Solution:** A robust fallback system.
If the logic in Step 4 returns fewer than 3 recommendations, the system automatically:
1.  Identifies the **most popular items** from the EDA (e.g., "Large Seasoned Fries," "20 Oz Soda").
2.  Recommends the top popular items that are *not already in the user's cart*.

This ensures that **every user receives a sensible recommendation 100% of the time**, preventing a negative user experience and still providing a high-likelihood upsell opportunity.

## 7. How to Run This Project

### Prerequisites
* [cite_start]Python 3.10+ (as per `mlxtend` compatibility, though 3.11 is preferred [cite: 66])
* Jupyter Notebook (or VS Code with Notebook support)

### Installation
1.  Clone this repository:
    ```bash
    git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
    cd your-repo-name
    ```
2.  Install the required packages:
    ```bash
    pip install -r requirements.txt
    ```

### Execution
1.  **`EDA.ipynb`**: This notebook contains all data cleaning, transformation, and exploratory analysis. It can be run to understand the data.
2.  **`WWT_REC.ipynb`**: This is the main file. Running all cells in this notebook will:
    * Load and process the data.
    * Train the Apriori model and generate association rules.
    * Apply the recommendation and fallback logic to the `test_data_question.csv` file.
    * [cite_start]Generate the final **`<TeamName>_Recommendation Output Sheet.xlsx`** file, ready for submission[cite: 61].

## 8. Answering the Client's Key Questions

This solution is designed to be presented to "Wings R Us" leadership, and it directly addresses their key concerns from the problem statement:

* [cite_start]**"What customer signals... should drive recommendations?"** [cite: 26]
    * **Our Answer:** The most powerful signal is **item co-occurrence** within a single transaction. Our model finds patterns in what items customers *actually* buy together.

* [cite_start]**"How can your solution personalize...?"** [cite: 27]
    * **Our Answer:** This model uses **cart-based personalization**. The recommendations are not static; they change in real-time based on the specific items a user adds to their cart.

* [cite_start]**"Recommendations should feel fresh..."** [cite: 28]
    * **Our Answer:** While our current model provides the *strongest* recommendation, we can introduce variety by sampling from the Top 10 rules instead of just taking the Top 3. This prevents a user from seeing the exact same recommendation every time.

* [cite_start]**"How should we define and measure success?"** [cite: 29]
    * **Our Answer:**
        1.  [cite_start]**Technical Metric:** Recall@3 (as used in this competition)[cite: 126].
        2.  [cite_start]**Business Metrics:** We will track **Average Order Value (AOV)** [cite: 23][cite_start], **Recommendation Conversion Rate** (what % of users add a recommended item), and **Items Per Basket**[cite: 8].

* [cite_start]**"How will you ensure consistency across platforms?"** [cite: 30]
    * **Our Answer:** This model's logic (the set of rules) will be deployed as a single microservice (API). The app, website, and kiosk will all call the same API, sending the cart contents and receiving the same recommendation. This ensures 100% consistency.

* [cite_start]**"Can you propose a fast, value-proving pilot?"** [cite: 31, 32]
    * **Our Answer:** Absolutely. We propose a 2-week **A/B Test**.
        * **Group A (Control):** 50% of users see the normal checkout.
        * **Group B (Treatment):** 50% of users see the new recommendations.
    * We will measure the AOV and Conversion Rate for both groups. [cite_start]If Group B shows a statistically significant increase in AOV, the pilot is a success, and the feature is proven to add value[cite: 32].

## 9. Future Improvements

* [cite_start]**Segmented Rules:** Train different recommendation models for different `CUSTOMER_TYPE` segments (e.g., "registered" vs. "guest")[cite: 50], as their buying habits may differ.
* [cite_start]**Contextual Rules:** Incorporate store location (`STATE`) [cite: 52] [cite_start]or order channel (`ORDER_CHANNEL_NAME`) [cite: 41] as contexts for recommendations.
* **Sequential Analysis:** Evolve from association rules to sequential pattern mining to recommend items based on a user's *past order history*, not just their current cart.