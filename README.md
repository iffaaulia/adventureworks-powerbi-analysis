# AdventureWorks Business Performance Analysis | Power BI

My second business analysis portfolio project explores how Power BI can turn raw business data into an interactive report for reviewing sales, products, customers, and returns.

This is a guided learning project completed through a Maven Analytics course. It documents my progress in Power BI and my first steps in interpreting business performance. It is not a client engagement, and the recommendations below have not been implemented or tested in a business.

## Business questions

- How are revenue, profit, orders, and returns changing over time?
- Which products and categories should be investigated further?
- How do selected products perform against monthly targets?
- How are orders distributed geographically and across customer segments?
- How does the report's price adjustment scenario affect calculated profit?

## Workflow and skills

### 1. Connecting and preparing CSV data

I connected CSV files in Power BI Desktop and used Power Query to prepare the data. I practiced profiling column quality, distribution, and values to investigate errors, empty values, duplicates, and minimum/maximum values. Profiling helped identify issues; I do not claim that every type of issue was present or removed.

I assigned appropriate column types, appended the 2020, 2021, and 2022 sales queries, and practiced deriving calendar attributes such as start of week, month, and quarter. I also created custom columns for calculations such as a discounted price. Power Query custom columns belong to the transformation layer and are distinct from DAX calculated columns.

### 2. Building the data model

I learned to distinguish fact and dimension tables, identify unique lookup keys and matching foreign keys, and use relationships to propagate filters without flattening every table through merges. In Power BI, these are model keys and relationships rather than database primary-key constraints.

The supplied Model view shows Sales Data and Returns Data as fact tables sharing Product, Calendar, and Territory dimensions. Customer Lookup relates to Sales Data. Product Categories Lookup connects to Product Subcategories Lookup, which connects to Product Lookup: a snowflake branch within the model. I practiced star and snowflake concepts, filter context and filter flow, and date and territory hierarchies.

![Data model](screenshots/data-model.png)

### 3. Calculated columns and DAX measures

I learned the difference between implicit aggregations and explicit measures, and between row-level calculated columns stored in the model and measures evaluated in filter context. The following expressions are transcribed from my screenshots:

```dax
% of All Orders =
DIVIDE([Total Orders], [All Orders])

Average Retail Price =
AVERAGE('Product Lookup'[ProductPrice])

Profit Target =
[Previous Month Profit] * 1.1

Return Rate =
DIVIDE([Quantity Returned], [Quantity Sold], "No Sales")

Total Customer =
DISTINCTCOUNT('Sales Data'[CustomerKey])
```

These examples practice safe division, averaging, measure composition, and distinct counts. Profit Target is defined as 110% of previous-month profit; it is a project calculation rather than a documented stakeholder-approved target. Average Retail Price is an unweighted average of product catalog prices in the current context, not average realized selling price. Total Customer counts distinct customer keys represented in sales in the current context.

Return Rate is a unit-based metric. Its supplied alternative result is the text `"No Sales"`; returning BLANK() instead could be considered if downstream numeric calculations require it. The formula above preserves the submitted implementation.

The measure list also shows rolling-window measures, previous-month comparisons, scenario measures, and target gaps. Their formulas have not all been supplied, so their names are evidence of implementation scope, not a full formula audit. The behavior of % of All Orders depends on how [All Orders] removes filters; its denominator definition remains to be confirmed.

### Source-data cross-check

A separate check of the uploaded CSVs found 56,046 sales rows across three yearly files, 25,164 distinct order numbers, 84,174 units sold, 1,828 returned units, and 17,416 distinct purchasing customer keys. The unit return ratio is approximately 2.17%, consistent with the rounded dashboard headline. This is an aggregate source-data comparison, not verification of every Power Query transformation or DAX measure.

Distinct order counts by category were Accessories 16,983, Bikes 13,929, and Clothing 6,976. They match the displayed rounded values. The same order can contain multiple categories, so these counts overlap and should not be summed as unique total orders.

### 4. Designing and building the interactive report

I started by planning a four-page layout, with each page supporting a different level of business exploration:

- **Executive dashboard:** summarize total revenue, profit, orders, and the unit return rate, then provide trends and product/category comparisons.
- **Map:** explore total orders by country, with continent selection for geographic filtering.
- **Product detail:** investigate a product selected from the executive dashboard's Top 10 product table through a product-detail navigation workflow.
- **Customer detail:** explore customer contribution and order distribution across income levels and occupations.

On the executive dashboard, I combined KPI cards with a line chart for revenue trends, a bar chart for orders by category, and a matrix/table for the Top 10 products. I added tooltips to provide more context during exploration and sidebar buttons to navigate between pages. These choices let users move from a summary to more detailed views within the same report.

I applied the same layout and navigation approach to the customer page, adding donut charts to compare orders across customer income and occupation groups. These charts show order distribution by customer attributes; they do not measure the number of customers in each group or establish segment profitability.

The product-detail page includes target comparisons, a price-adjustment scenario, and a metric selector. The category tooltip is a supporting report page in addition to the four main user-facing pages.

This stage helped me practice choosing visuals around a question, designing a consistent multi-page experience, and adding contextual information without placing every metric on the main canvas. The screenshots demonstrate selected report states. The precise interaction used to open Product Detail should be confirmed in Power BI Desktop before documenting it as a single-click action or a specific drill-through gesture.

### 5. Product targets and parameter-driven exploration

On Product Detail, I learned to use gauge charts to compare monthly orders, revenue, and profit with their respective targets. This helps users see the selected product's performance against a benchmark. The supplied Profit Target expression uses previous-month profit multiplied by 1.1; order and revenue target formulas still need to be documented.

For the bottom chart, I created a field parameter through New parameter > Fields. It lets the user switch one visual between Orders, Revenue, Profit, Returns, and Return Rate. This reduces repeated charts while supporting different analytical questions. Titles, formatting, and units should follow the selected metric, especially when switching between counts, currency, and percentages.

For the middle chart, I used a numeric range parameter to explore price adjustments and compare baseline profit with adjusted profit. This is a what-if input consumed by scenario measures, whereas the field parameter changes the measure displayed. The scenario is not a forecast of customer demand: its pricing, cost, and quantity assumptions must be documented from the underlying DAX before recommending a price change.

## Report pages

| Page | Purpose | Features visible in the report |
| --- | --- | --- |
| Executive dashboard | Review overall performance | KPI cards, revenue trend, category comparison, product table, monthly comparisons |
| Map | Explore geographic order distribution | Country bubbles and continent selection |
| Product detail | Investigate selected product performance | Monthly target gauges, price adjustment scenario, metric selector |
| Customer detail | Explore customers and their contribution | Customer trend, revenue per customer, segment charts, top customer table |
| Category tooltip | Provide context during exploration | Revenue, profit, orders, returns, and weekly order trend |

![Executive dashboard with revenue trend tooltip](screenshots/exec-dashboard.png)

![Category report tooltip](screenshots/category-tooltip.png)

![Geographic view](screenshots/map.png)

![Product detail](screenshots/product-detail.png)

![Customer detail](screenshots/customer-detail.png)

## Business findings and recommendations

### Basis of analysis

The uploaded sales CSVs cover January 1, 2020 through June 30, 2022. A separate calculation joins sales product keys to the product lookup and computes revenue as quantity multiplied by catalog ProductPrice, and product profit as quantity multiplied by ProductPrice minus ProductCost. These totals reconcile to the rounded dashboard KPIs. Profit here is a product-price-minus-cost measure, not verified net income after operating expenses, taxes, discounts, or return adjustments.

Overall, the source data yields $24,914,586.82 revenue, $10,457,715.43 product profit, 25,164 distinct orders, and 17,416 purchasing customers. Returned units divided by units sold is 1,828 / 84,174 = 2.17%. Returns are aggregated from the separate returns dataset; this ratio is not a cohort-matched probability that a particular order will be returned.

### 1. Higher revenue per order supported June revenue growth

June 2022 revenue was $1,826,987.14 versus $1,768,432.51 in May, an increase of 3.31%. Distinct orders fell from 2,165 to 2,146 (-0.88%). Revenue per order increased from $816.83 to $851.35 (+4.23%).

**Interpretation:** June's revenue growth coincided with a higher average order value, despite fewer orders. The data summarized here does not establish whether product mix, quantities per basket, or pricing caused the increase.

**Recommendation:** Decompose the change by product category, units per order, and product-level contribution before choosing a sales or pricing action. Track revenue per order alongside order count.

### 2. Category order popularity and revenue contribution differ

Accessories appeared in 16,983 distinct orders, compared with 13,929 for Bikes. Yet Bikes contributed $23,642,495.10, approximately 94.89% of revenue; Accessories contributed $906,673.11. Category order counts overlap because a basket can contain multiple categories.

**Interpretation:** A category with more orders is not necessarily the main revenue driver. Revenue is highly concentrated in Bikes.

**Recommendation:** Track Bikes revenue, margin, and availability closely. Test accessory bundles with relevant bike purchases and evaluate incremental profit and attachment rate. The current data does not prove that bundling will improve results.

### 3. Two helmet variants warrant a return review

Sport-100 Helmet, Red recorded 70 returned units against 2,099 sold (3.33%); Blue recorded 66 against 1,995 (3.31%). Both exceed the aggregate unit return ratio of 2.17%.

**Interpretation:** These variants are candidates for investigation. The comparison does not establish a defect or explain the reason for returns.

**Recommendation:** Review trends by period and territory, then obtain return reasons, size/fit information, or quality records if available. Assess unit volume and financial materiality before prioritizing an intervention. Return reasons are not present in the supplied returns CSV.

### 4. Order volume and order value provide different geographic signals

The United States had 8,700 orders and approximately $7.94M revenue. Australia had 6,060 orders and approximately $7.42M revenue. Australia generated similar revenue with fewer orders, implying a higher revenue per order across the full dataset.

**Recommendation:** Add revenue and revenue per order to the geographic analysis and examine product mix before comparing market performance or allocating marketing budget. The map currently encodes total orders and cannot alone establish market profitability or demand potential.

### 5. Customer and product views support targeted follow-up

The report shows approximately 17.4K purchasing customers and $1.43K revenue per customer. Its customer page supports exploration of high-contributing customers and orders by income level and occupation. The product page adds monthly target comparisons and scenario exploration.

**Recommendation:** Investigate repeat purchase behavior and customer revenue concentration before proposing a retention program. Use product target gaps to prioritize investigation, while documenting the target rules. Use the numeric range scenario to frame questions about price sensitivity rather than claim a realized or predicted profit increase.

## Learning reflection

This project helped me practice the full Power BI workflow, from preparing data and understanding data models to creating calculations and presenting results through interactive visuals. It also introduced a new challenge: moving from describing a chart to asking what the result means for a business and what evidence is needed before recommending an action.

## Scope and limitations

- The implementation followed a Maven Analytics course. No independent dashboard extensions are claimed here; any confirmed additions will be documented separately.
- Findings combine screenshot evidence with independent aggregate checks of the supplied CSVs. Recommendations are proposed follow-up actions.
- Five DAX examples and the Model view have been reviewed from screenshots. Full relationship properties, dependent measures, and Power Query Applied Steps remain to be checked.
- CSV checks confirm overlapping distinct order counts across categories. The Total Orders expression itself still needs to be supplied to verify its exact implementation.
- The price adjustment visual is a calculation scenario. Its DAX assumptions must be documented before interpreting it as a business forecast; the screenshot does not demonstrate demand elasticity.
- The screenshots demonstrate report states, but interactive behavior still needs to be checked in Power BI Desktop.
- Business recommendations are proposals; no realized revenue increase, cost saving, or stakeholder impact is claimed.

## Open the report

Open `AdventureWorks Report.pbix` in Power BI Desktop. Inspect the report before refreshing; source file paths may need to be updated to the original course dataset location. The raw source files are not included in this package.

## Acknowledgment

Maven Analytics provided the course and AdventureWorks learning materials used for this guided project. This portfolio entry describes my learning process and initial interpretation of the report.
