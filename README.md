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
