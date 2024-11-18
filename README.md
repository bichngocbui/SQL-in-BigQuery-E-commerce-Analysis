# E-commerce Analytics with BigQuery
## Project Overview
This project demonstrates advanced data analysis of e-commerce behavior using Google BigQuery, focusing on user interactions, purchasing patterns, and product performance metrics. The analysis encompasses various aspects of the e-commerce journey, from traffic sources to purchase completion, providing valuable insights for business decision-making.
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
## Business Value
This analysis provides actionable insights for:
- Marketing channel effectiveness
- User behavior patterns
- Product performance optimization
- Conversion funnel optimization
- Customer purchase patterns
- Revenue optimization opportunities
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
## Technical Implementation
The project leverages BigQuery's powerful features including:
- Complex data structures handling through UNNEST operations
- Advanced SQL aggregations and window functions
- Cohort analysis implementation
- Custom metrics calculations (bounce rates, conversion rates, etc.)
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
![image](https://github.com/user-attachments/assets/adcb33ec-d651-4b6a-826e-1fc28f202507)

This table provides a clear summary of total ___visits, pageviews, and transactions___ for the ___first quarter of 2017___. ___January and March___ show notably ___higher engagement___, with ___March___ reaching the ___highest___ numbers across all metrics, particularly transactions (993). ___February___ reflects ___a slight dip___, possibly indicating ___seasonal or user behavior fluctuations___. Overall, this data effectively highlights monthly trends in user activity and transaction volume.
### Bounce rate per traffic source in July 2017
#### Syntax
```sql 
SELECT
    trafficSource.source as source,
    sum(totals.visits) as total_visits,
    sum(totals.Bounces) as total_no_of_bounces,
    round (sum (totals.bounces)/sum (totals.visits) * 100, 1) as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visits DESC;
```
#### Result 
![image](https://github.com/user-attachments/assets/41986ec7-085b-4fb4-9088-b65910c7fede)

The ___bounce rate___ varies significantly across sources. Major traffic sources like ___Google (51.6%)__ and ___(direct) (43.3%)___ have ___moderate__ bounce rates, indicating relatively engaged users. However, some sources, such as ___l.facebook.com (88.2%)___, ___duckduckgo.com (87.5%)__, and __search.mysearch.com (91.7%)___, show ___very high___ bounce rates, suggesting lower user engagement. On the other hand, sources like ___plus.google.com (25%)___ and ___hangouts.google.com (20%)___ exhibit ___much lower___ bounce rates, reflecting higher interaction levels.
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
)

select * from month_data
union all
select * from week_data
order by time_type, revenue DESC;
```
#### Result
![image](https://github.com/user-attachments/assets/058fa44c-5fb4-4715-b65a-d0536a309280)

The table shows ___revenue data___ by ___month___ and ___week___ from various ___sources___. In ___June 2017___, the top revenue source was ___(direct)___ with 97,333.62, followed by ___google___ with 18,757.18. Weekly data reveals that ___(direct)___ consistently ___performs best___, with a peak in ___Week 201724___ at 30,908.91, while ___google___ maintains ___steady contributions__. Sources like ___phandroid.com___ and ___sites.google.com___ have lower revenue, suggesting areas for improvement. Overall, the data highlights key revenue drivers like ___(direct)__ and ___google___, and shows opportunities for optimizing other sources.
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
![image](https://github.com/user-attachments/assets/47665165-32fc-458d-ba35-a256c3dcb06f)

The table shows that ___average pageviews___ for ___purchase sessions___ ___increased___ from June (94.02) to July (124.24), indicating ___higher engagement___ in purchasing activities in July. However, ___non-purchase sessions___ consistently have a ___higher___ average pageview count compared to ___purchase sessions___, with the count increasing from June (316.87) to July (334.06). This trend might highlight opportunities to ___improve conversion strategies for non-purchase sessions___.
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
![image](https://github.com/user-attachments/assets/9b82e41f-e3b1-4be2-a01c-5ad2493a0d9d)

In ___July 2017___, the ___average number of transactions per user___ was approximately ___4.16___. This metric can provide insight into ___user activity levels___ and ___transaction frequency___, potentially useful for ___tracking engagement___ or ___setting benchmarks___ for future months.
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
![image](https://github.com/user-attachments/assets/9103db6f-4e6d-4f6c-aefa-a63dfbd26375)

In ___July 2017___, the ___average revenue___ generated ___by user per visit___ was approximately ___43.86___. This metric provides insight into the ____revenue efficiency per visit___, which can help assess user spending behavior and guide strategies to optimize revenue per visit.
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
![image](https://github.com/user-attachments/assets/4912a796-3ebb-475a-8ccd-89b3f651ce16)

This data lists ___other products purchased by customers who bought the "YouTube Men's Vintage Henley"___ in ___July 2017___. The ___"Google Sunglasses"___ stands out as ___the most frequently purchased___ additional item, with a quantity of 20, followed by ___the "Google Women's Vintage Hero Tee Black"___ with 7 purchases and ___"SPF-15 Slim & Slender Lip Balm"___ with 6 purchases. ___Most other products___ have ___lower quantities___, with the majority purchased just once or twice, highlighting a wide variety of additional interests among these customers.
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
![image](https://github.com/user-attachments/assets/263d5f22-1e5f-4c15-a1c4-c5f0b74c49cd)

The data shows the ___performance metrics for product engagement___ from ___January to March 2017___. Overall, there is a ___positive trend___ in both the ___add-to-cart rate___ and ___purchase rate___ over the three months, with ___March___ showing the ___highest__ values at 37.29% and 12.64%, respectively. This indicates ___increasing customer interest and conversion effectiveness___. Notably, ____the number of product views fluctuates___, suggesting that ____marketing efforts___ or ___seasonal factors___ may influence customer engagement.
## Conclusion
This project has provided valuable insights into consumer behavior in the e-commerce landscape through data analysis using Google Analytics on BigQuery. Key findings include:
- User Engagement: A noticeable increase in page views and transactions, particularly in March 2017, suggests improved user engagement. Ongoing optimization of marketing channels is essential.
- Traffic Source Analysis: Direct traffic has a lower bounce rate compared to other sources, indicating higher user interest. Sources with high bounce rates should be examined for content improvements.
- Product Performance: Variations in revenue and conversion rates among products highlight the need to optimize underperforming items to enhance overall sales.
- Shopping Behavior Patterns: Consumers are increasingly making multiple purchases per session, indicating a positive trend in spending habits.
- Optimization Opportunities: Continuous monitoring of high-performing traffic sources and addressing conversion weaknesses can significantly boost revenue and customer satisfaction.

Overall, these insights lay a foundation for strategic marketing and product development efforts, emphasizing the importance of ongoing data analysis for sustained success in e-commerce.
