# E-commerce Analytics with BigQuery
## Project Overview
This project demonstrates advanced data analysis of e-commerce behavior using Google BigQuery, focusing on user interactions, purchasing patterns, and product performance metrics. The analysis encompasses various aspects of the e-commerce journey, from traffic sources to purchase completion, providing valuable insights for business decision-making.
## Data access & Structure 
### Dataset information 
The analysis utilizes the Google Analytics sample dataset available in BigQuery:
- Dataset: `bigquery-public-data.google_analytics_sample`
- Table: ga_sessions_*
- Time period: 2017
### Data Schema
For detailed information about the Google Analytics dataset schema, please refer to the official Google Analytics documentation, available at at https://support.google.com/analytics/answer/3437719?hl=en ↩
### Data Access Patterns
Key data access patterns include:
- Basic session data: Direct access to ga_sessions_* table
- Product data: Requires UNNEST(hits) and UNNEST(hits.product)
- E-commerce actions: Requires UNNEST(hits) to access eCommerceAction
- Revenue data: Accessed through product.productRevenue after unnesting
## Key Analysis Areas
### Traffic and Engagement Analysis
- Comprehensive tracking of visits, pageviews, and transactions across different time periods
- Bounce rate analysis by traffic source to understand user engagement quality
- Deep dive into pageview patterns comparing purchasers vs. non-purchasers
### Revenue Analysis
- Multi-dimensional revenue tracking by traffic source, broken down by week and month
- Average transaction values and session spending patterns
- Customer purchase behavior and revenue generation metrics
### Product Performance
- Detailed product-level analysis including cross-selling patterns
- Product view to purchase conversion funnel analysis
- Cohort mapping from product view to add-to-cart to purchase completion
## Technical Implementation
The project leverages BigQuery's powerful features including:
- Complex data structures handling through UNNEST operations
- Advanced SQL aggregations and window functions
- Cohort analysis implementation
- Custom metrics calculations (bounce rates, conversion rates, etc.)
## Business Value
This analysis provides actionable insights for:
- Marketing channel effectiveness
- User behavior patterns
- Product performance optimization
- Conversion funnel optimization
- Customer purchase patterns
- Revenue optimization opportunities
## Exploring the dataset 
### Query 1: Calculate total visit, pageview, transaction for Jan, Feb and March 2017
#### Syntax 
```sql
SELECT 
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
    SUM(totals.visits) AS visits, 
    SUM(totals.pageviews) AS pageviews,   
    SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
#### Result
| month   | visits | pageviews | transactions |
|---------|--------|-----------|--------------|
| 201701  | 64694  | 257708    | 713         |
| 201702  | 62192  | 233373    | 733         |
| 201703  | 69931  | 259522    | 993         |

This table provides a clear summary of total visits, pageviews, and transactions for the first quarter of 2017. January and March show notably higher engagement, with March reaching the highest numbers across all metrics, particularly transactions (993). February reflects a slight dip, possibly indicating seasonal or user behavior fluctuations. Overall, this data effectively highlights monthly trends in user activity and transaction volume.
### Bounce rate per traffic source in July 2017
#### Syntax
```sql 
SELECT
    trafficSource.source as source,
    sum(totals.visits) as total_visits,
    sum(totals.Bounces) as total_no_of_bounces,
    (sum(totals.Bounces)/sum(totals.visits))* 100 as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visits DESC;
```
#### Result 
| Row | Source                   | Total Visits | Total No. of Bounces | Bounce Rate (%)     |
|-----|--------------------------|--------------|-----------------------|----------------------|
| 2   | (direct)                | 19,891       | 8,606                 | 43.266               |
| 3   | youtube.com              | 6,351        | 4,238                 | 66.730               |
| 4   | analytics.google.com     | 1,972        | 1,064                 | 53.955               |
| 5   | Partners                 | 1,788        | 936                   | 52.349               |
| 6   | m.facebook.com           | 669          | 430                   | 64.275               |
| 7   | google.com               | 368          | 183                   | 49.728               |
| 8   | dfa                      | 302          | 124                   | 41.060               |
| 9   | sites.google.com         | 230          | 97                    | 42.174               |
| 10  | facebook.com             | 191          | 102                   | 53.403               |

The table presents a detailed summary of traffic sources, showing total visits, bounce numbers, and bounce rates. Notably, youtube.com exhibits the highest bounce rate at 66.730%, indicating potential challenges in keeping users engaged. Conversely, (direct) traffic has a lower bounce rate of 43.266%, suggesting that users are more likely to stay engaged when accessing the site directly. The high total visits from google at 38,400 reveal its significance as a primary traffic source, despite a moderate bounce rate of 51.557%. This data can help optimize user experience and improve retention strategies.
### Revenue by traffic source by week, by month in June 2017
#### Syntax
```sql
with 
month_data as(
  SELECT
    "Month" as time_type,
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  order by revenue DESC
),

week_data as(
  SELECT
    "Week" as time_type,
    format_date("%Y%W", parse_date("%Y%m%d", date)) as week,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  order by revenue DESC
)

select * from month_data
union all
select * from week_data
order by time_type;
```
#### Result
| Row | time_type |   month  |          source          |   revenue   |
|-----|-----------|----------|--------------------------|-------------|
|  1  |   Month   | 201706   | (direct)                 | 97333.62    |
|  2  |   Month   | 201706   | google                   | 18757.18    |
|  3  |   Month   | 201706   | phandroid.com            |   52.95     |
|  4  |   Month   | 201706   | dealspotr.com            |   72.95     |
|  5  |   Month   | 201706   | sites.google.com         |   39.17     |
|...  | ...       | ...      | ...                      |...          |
| 17  |   Week    | 201724   | (direct)                 | 30908.91    |
| 18  |   Week    | 201724   | mail.google.com          |  2486.86    |
| 19  |   Week    | 201722   | (direct)                 |  6888.90    |
| 20  |   Week    | 201725   | phandroid.com            |   52.95     |
| 21  |   Week    | 201723   | (direct)                 | 17325.68    |

The table displays revenue data categorized by month and week from various sources. Notably, the month of June 2017 (201706) records a total revenue of 97,333.62, with significant contributions from direct traffic and Google. Weekly data shows fluctuations in revenue, indicating varying performance across different sources.
Some sources, such as "phandroid.com" and "sites.google.com," contribute relatively low revenue compared to leading sources, suggesting potential opportunities for improving engagement or marketing strategies. Overall, this data provides valuable insights for trend analysis and optimizing resource allocation in marketing efforts.
### Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
#### Syntax
```sql
with 
purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      (sum(totals.pageviews)/count(distinct fullvisitorid)) as avg_pageviews_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions>=1
  and product.productRevenue is not null
  group by month
),

non_purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      sum(totals.pageviews)/count(distinct fullvisitorid) as avg_pageviews_non_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions is null
  and product.productRevenue is null
  group by month
)

select
    pd.*,
    avg_pageviews_non_purchase
from purchaser_data pd
full join non_purchaser_data using(month)
order by pd.month;
```
#### Result 
| Row |  month  | avg_pageviews_purchase | avg_pageviews_non_purchase |
|-----|---------|------------------------|----------------------------|
|  1  | 201706  | 94.02                  | 316.87                     |
|  2  | 201707  | 124.24                 | 334.06                     |

The table shows that average pageviews for purchase sessions increased from June (94.02) to July (124.24), indicating higher engagement in purchasing activities in July. However, on-purchase sessions consistently have a higher average pageview count compared to purchase sessions, with the count increasing from June (316.87) to July (334.06). This trend might highlight opportunities to improve conversion strategies for non-purchase sessions.
### Average number of transactions per user that made a purchase in July 2017
#### Syntax
```sql
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    sum(totals.transactions)/count(distinct fullvisitorid) as Avg_total_transactions_per_user
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,unnest (hits) hits,
    unnest(product) product
where  totals.transactions>=1
and product.productRevenue is not null
group by month;
```
### Result 
| Row |  month  | Avg_total_transactions_per_user |
|-----|---------|---------------------------------|
|  1  | 201707  |           4.16                  |

In July 2017, the average number of transactions per user was approximately 4.16. This metric can provide insight into user activity levels and transaction frequency, potentially useful for tracking engagement or setting benchmarks for future months.
### Average amount of money spent per session. Only include purchaser data in July 2017
#### Syntax
``` sql
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    ((sum(product.productRevenue)/sum(totals.visits))/power(10,6)) as avg_revenue_by_user_per_visit
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,unnest(hits) hits
  ,unnest(product) product
where product.productRevenue is not null
and totals.transactions>=1
group by month;
```
#### Result 
| Row |  month  | avg_revenue_by_user_per_visit |
|-----|---------|-------------------------------|
|  1  | 201707  |           43.86               |

In July 2017, the average revenue generated per user per visit was approximately 43.86. This metric provides insight into the revenue efficiency per visit, which can help assess user spending behavior and guide strategies to optimize revenue per visit.
### Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017
#### Syntax
``` sql
with buyer_list as(
    SELECT
        distinct fullVisitorId
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue is not null
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```
#### Result 
| Row | other_purchased_products                   | quantity |
|-----|--------------------------------------------|----------|
|  1  | Google Sunglasses                          | 20       |
|  2  | Google Women's Vintage Hero Tee Black      | 7        |
|  3  | SPF-15 Slim & Slender Lip Balm             | 6        |
|  4  | Google Women's Short Sleeve Hero Tee Red Heather | 4   |
|  5  | YouTube Men's Fleece Hoodie Black          | 3        |
|  6  | Google Men's Short Sleeve Badge Tee Charcoal | 3      |
|  7  | Android Wool Heather Cap Heather/Black     | 2        |
|  8  | Crunch Noise Dog Toy                       | 2        |
|  9  | Android Men's Vintage Henley               | 2        |
| 10  | 22 oz YouTube Bottle Infuser               | 2        |

This data lists other products purchased by customers who bought the "YouTube Men's Vintage Henley" in July 2017. The "Google Sunglasses" stands out as the most frequently purchased additional item, with a quantity of 20, followed by the "Google Women's Vintage Hero Tee Black" with 7 purchases and "SPF-15 Slim & Slender Lip Balm" with 6 purchases. Most other products have lower quantities, with the majority purchased just once or twice, highlighting a wide variety of additional interests among these customers.
### Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017
#### Syntax
```sql
with product_data as(
select
    format_date('%Y%m', parse_date('%Y%m%d',date)) as month,
    count(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) as num_product_view,
    count(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) as num_add_to_cart,
    count(CASE WHEN eCommerceAction.action_type = '6' and product.productRevenue is not null THEN product.v2ProductName END) as num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,UNNEST(hits) as hits
,UNNEST (hits.product) as product
where _table_suffix between '20170101' and '20170331'
and eCommerceAction.action_type in ('2','3','6')
group by month
order by month
)

select
    *,
    round(num_add_to_cart/num_product_view * 100, 2) as add_to_cart_rate,
    round(num_purchase/num_product_view * 100, 2) as purchase_rate
from product_data;
```
#### Result 
| Row | month   | num_product_view | num_add_to_cart | num_purchase | add_to_cart_rate | purchase_rate |
|-----|---------|------------------|------------------|--------------|-------------------|----------------|
|  1  | 201701  | 25787            | 7342             | 2143         | 28.47             | 8.31           |
|  2  | 201702  | 21489            | 7360             | 2060         | 34.25             | 9.59           |
|  3  | 201703  | 23549            | 8782             | 2977         | 37.29             | 12.64          |

The data shows the performance metrics for product engagement from January to March 2017. Overall, there is a positive trend in both the add-to-cart rate and purchase rate over the three months, with March showing the highest values at 37.29% and 12.64%, respectively. This indicates increasing customer interest and conversion effectiveness. Notably, the number of product views fluctuates, suggesting that marketing efforts or seasonal factors may influence customer engagement.
## Conclusion
This project has provided valuable insights into consumer behavior in the e-commerce landscape through data analysis using Google Analytics on BigQuery. Key findings include:
- User Engagement: A noticeable increase in page views and transactions, particularly in March 2017, suggests improved user engagement. Ongoing optimization of marketing channels is essential.
- Traffic Source Analysis: Direct traffic has a lower bounce rate compared to other sources, indicating higher user interest. Sources with high bounce rates should be examined for content improvements.
- Product Performance: Variations in revenue and conversion rates among products highlight the need to optimize underperforming items to enhance overall sales.
- Shopping Behavior Patterns: Consumers are increasingly making multiple purchases per session, indicating a positive trend in spending habits.
- Optimization Opportunities: Continuous monitoring of high-performing traffic sources and addressing conversion weaknesses can significantly boost revenue and customer satisfaction.

Overall, these insights lay a foundation for strategic marketing and product development efforts, emphasizing the importance of ongoing data analysis for sustained success in e-commerce.
