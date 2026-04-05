# [SQL-Big Query] Website Performance Analysis in eCommerce 
## 1. Business requirements

The project was conducted to assist **Sales and Marketing Manager** to have an outlook on the business situation and marketing efficiency by analysing performance of the website, in terms of **Revenue, Bounce rate, The patterns of user behaviour, Products, Customer journey, etc**. 


Dataset: The Google Analytics (GA) Sample Dataset record of every user interaction on the Google Merchandise Store from 2016 to 2017. This dataset provides the raw digital track of every visitor—including their traffic source, device type, and the exact sequence of pageviews and clicks leading to a purchase.

## 2. Dataset understanding
<details>
<summary> Data dictionary </summary>

| Field Name               | Data Type | Description                                                                                                                                                                                                                                                                                                                                                                                |
|--------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| clientId                 | STRING    | Unhashed version of the Client ID for a given user associated with any given visit/session.                                                                                                                                                                                                                                                                                                |
| fullVisitorId            | STRING    | The unique visitor ID.                                                                                                                                                                                                                                                                                                                                                                     |
| visitorId                | NULL      | This field is deprecated. Use "fullVisitorId" instead.                                                                                                                                                                                                                                                                                                                                     |
| userId                   | STRING    | Overridden User ID sent to Analytics.                                                                                                                                                                                                                                                                                                                                                      |
| visitNumber              | INTEGER   | The session number for this user. If this is the first session, then this is set to 1.                                                                                                                                                                                                                                                                                                     |
| visitId                  | INTEGER   | An identifier for this session. This is part of the value usually stored as the _utmb cookie. This is only unique to the user. For a completely unique ID, you should use a combination of fullVisitorId and visitId.                                                                                                                                                                      |
| visitStartTime           | INTEGER   | The timestamp (expressed as POSIX time).                                                                                                                                                                                                                                                                                                                                                   |
| date                     | STRING    | The date of the session in YYYYMMDD format.                                                                                                                                                                                                                                                                                                                                                |
| totals                   | RECORD    | This section contains aggregate values across the session.                                                                                                                                                                                                                                                                                                                                 |
| totals.bounces           | INTEGER   | Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.                                                                                                                                                                                                                                                                                              |
| totals.hits              | INTEGER   | Total number of hits within the session.                                                                                                                                                                                                                                                                                                                                                   |
| totals.newVisits         | INTEGER   | Total number of new users in session (for convenience). If this is the first visit, this value is 1, otherwise it is null.                                                                                                                                                                                                                                                                 |
| totals.pageviews         | INTEGER   | Total number of pageviews within the session.                                                                                                                                                                                                                                                                                                                                              |
| totals.screenviews       | INTEGER   | Total number of screenviews within the session.                                                                                                                                                                                                                                                                                                                                            |
| totals.sessionQualityDim | INTEGER   | An estimate of how close a particular session was to transacting, ranging from 1 to 100, calculated for each session. A value closer to 1 indicates a low session quality, or far from transacting, while a value closer to 100 indicates a high session quality, or very close to transacting. A value of 0 indicates that Session Quality is not calculated for the selected time range. |

</details>

## 3. Data analysing
### Query 1: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
```sql
SELECT 
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) month_extract
    ,SUM(totals.visits) visits
    ,SUM(totals.pageviews) pageviews
    ,SUM(totals.transactions) transactions
    ,ROUND(SUM(totals.totalTransactionRevenue)/POW(10,6),2) revenue
   FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
   WHERE _table_suffix BETWEEN '0101' AND '0331'
   GROUP BY month_extract
```
| month  | visits | pageviews | transactions | revenue   |
|--------|--------|-----------|--------------|-----------|
| 201701 | 64694  | 257708    | 713          | 106248.15 |
| 201702 | 62192  | 233373    | 733          | 116111.6  |
| 201703 | 69931  | 259522    | 993          | 150224.7  |

The table provides the overview of the website performance across three first months in 2017, indicating:

- **Revenue Growth:** Revenue surged by 41.4% ($106k to $150k) over three months, significantly outpacing the 8% growth in traffic.
  
- **Transaction Spike:** March saw a major performance jump, with transactions increasing by 35.5% month-over-month despite only a 12.4% rise in visits.
  
- **Conversion Efficiency:** The disparity between modest traffic growth and rapid revenue gains indicates a high-quality user base and improved conversion efficiency toward the end of the quarter.
  
### Query 2. Bounce rate per traffic source (order by total_visit DESC)
```sql
select 
    trafficSource.source as source,
    sum(totals.visits) as total_visits, 
    sum(totals.bounces) as total_no_of_bounces, 
    sum(totals.bounces)/sum(totals.visits)*100 as bounce_rate

from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
group by trafficSource.source
order by total_visits desc
```
| source                      | total_visits | total_no_of_bounces | bounce_rate |
|-----------------------------|--------------|---------------------|-------------|
| google                      | 38400        | 19798               | 51.56       |
| (direct)                    | 19891        | 8606                | 43.27       |
| youtube.com                 | 6351         | 4238                | 66.73       |
| analytics.google.com        | 1972         | 1064                | 53.96       |
| Partners                    | 1788         | 936                 | 52.35       |
| m.facebook.com              | 669          | 430                 | 64.28       |
| google.com                  | 368          | 183                 | 49.73       |
| dfa                         | 302          | 124                 | 41.06       |


This analysis focuses on how different traffic sources influence user engagement, measured primarily through Bounce Rate (the % of sessions with no further action).

Key takeaways: 
- **High-Volume Sources Show Moderate Engagement:**
The primary drivers of traffic—Google, Direct, and YouTube—exhibit "average" engagement with bounce rates ranging between 40% and 65%. While these sources bring the most users, over half of them leave after viewing only one page. This suggests the landing pages are functional but may not be highly optimized for deep exploration.
- **High-Intent "Niche" Sources Perform Best:**
Platforms like Reddit and Mail.google.com achieved the lowest bounce rates (20–30%). Users arriving from these specific channels are significantly more engaged. This likely points to higher "content-to-user" relevancy or the success of targeted email marketing campaigns compared to broad search traffic.
- **Social Media Friction (The Mobile Gap):**
There is a notable discrepancy in Facebook's performance: Desktop users (Facebook.com) bounce at 53.4%, while mobile users (m.facebook.com) bounce much higher at 64.28%. The 11% gap indicates a potential mobile user experience (UX) issue. Content that works on desktop may be failing to engage users on mobile devices, or the mobile landing page load times are causing users to drop off.


### Query 3. Revenue by traffic source by week, by month in June 2017
```sql
select 
    "month" as time_type,
    format_date("%Y%m", parse_date ("%Y%m%d",date)) as month,  
    trafficSource.source as source,
    sum(totals.totalTransactionRevenue)/pow(10,6) as revenue
from `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
group by source, month
having sum(totals.totalTransactionRevenue) is not null

union all

select 
  "week"as time_type,
  format_date("%Y%W", parse_date ("%Y%m%d",date)) as week,  
  trafficSource.source as source,
  sum(totals.totalTransactionRevenue)/pow(10,6) as revenue
from `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
group by source, week
having sum(totals.totalTransactionRevenue) is not null
order by revenue desc
```
| time_type | month  | source            | revenue   |
|-----------|--------|-------------------|-----------|
| month     | 201706 | (direct)          | 97,231.62 |
| week      | 201724 | (direct)          | 30,883.91 |
| week      | 201725 | (direct)          | 27,254.32 |
| month     | 201706 | google            | 18,757.18 |
| week      | 201723 | (direct)          | 17,302.68 |
| week      | 201726 | (direct)          | 14,905.81 |
| week      | 201724 | google            | 9,217.17  |
| month     | 201706 | dfa               | 8,841.23  |

The revenue data highlights a massive disparity between "intent-driven" traffic (Direct, Google) and "browsing" traffic (Social).

Key takeaways:
- **Dominance of Direct Traffic:** Direct visits are the primary engine, generating $194,463 in June alone. This suggests extremely high brand equity and a loyal user base that bypasses search engines entirely.
- **Search & Display Synergy:** Google Search ($37.5k) and DoubleClick/DFA ($17.6k) serve as the secondary tier. Their combined performance validates the effectiveness of current SEO and paid display strategies in capturing new intent.
- **The "Social Gap":** While YouTube and Facebook drive high traffic, they generate almost no revenue. Their high bounce rates confirm that we are attracting 'window shoppers' rather than actual buyers.

### Query 4. Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
```sql
with sub1 as(
select 
  format_date("%Y%m", parse_date ("%Y%m%d",date)) as month,
  sum(totals.pageviews)/count(distinct fullVisitorId) as avg_pageviews_purchase
from `bigquery-public-data.google_analytics_sample.ga_sessions_*`
where _table_suffix between '20170601' and '20170731'
and  totals.transactions >=1
group by month
order by month),

sub2 as(
select format_date("%Y%m", parse_date ("%Y%m%d",date)) as month,
       sum(totals.pageviews)/count(distinct fullVisitorId) as avg_pageviews_non_purchase
from `bigquery-public-data.google_analytics_sample.ga_sessions_*`
where _table_suffix between '20170601' and '20170731'
and  totals.transactions is null
group by month
order by month)

select sub1.month, sub1.avg_pageviews_purchase, sub2. avg_pageviews_non_purchase
from sub1
inner join sub2
on sub1.month=sub2.month
order by month
```
| month  | avg_pageviews_purchase | avg_pageviews_non_purchase |
|--------|------------------------|----------------------------|
| 201706 | 25.73                  | 4.07                       |
| 201707 | 27.72                  | 4.19                       |

The table demonstrated behaviour of users who were purchaser and non-purchaser in 2 consecutive months (June and July, 2017). 

Key takeaways: 
- **The Engagement Gap:** There is a massive disparity in behavior between segments; purchasers average 25 pageviews per session, nearly 6x higher than non-purchasers (4 pageviews).
- **High-Intent Research:** The high pageview count for buyers suggests a "research-heavy" journey. These users aren't just browsing; they are actively comparing specs, reading reviews, and vetting details before committing to a transaction.
- **Optimization Strategy:** To capitalize on this, the UI should prioritize deep-content accessibility—such as visible reviews and side-by-side comparisons—to streamline the "deep dive" phase for high-intent users.

### Query 5. Average number of transactions per user that made a purchase in July 2017
```sql
SELECT 
  format_date("%Y%m", parse_date ("%Y%m%d",date)) as month,
  sum(totals.transactions)/ count (distinct fullvisitorID) as Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
WHERE totals.transactions >=1
GROUP BY month
```
| Month  | Avg_total_transactions_per_user |
|--------|---------------------------------|
| 201707 | 1.11                            |

This single data point gives us a baseline for User Frequency, which is a key measure of how often a customer returns to your funnel within a 30-day window.

Key takeaways: 

- **Baseline Engagement:** An average of 1.11 transactions per user indicates that the vast majority of your customers are "one-and-done" buyers within this specific month.
- **The "Power User" Gap:** Since the average is so close to 1.0, it suggests a lack of high-frequency repeat purchasers. To move this number to 1.5 or 2.0, you would likely need a dedicated retention strategy (like loyalty rewards or replenishment reminders).

### Query 6. Average amount of money spent per session. Only include purchaser data in July 2017
```sql

SELECT 
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
  ROUND((SUM(product.productRevenue) / SUM(totals.visits))/1000000,2) AS Avg_revenue_by_user_per_visit
FROM 
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`, 
  UNNEST(hits) AS hits, 
  UNNEST(hits.product) AS product
WHERE 
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170731'
  AND product.productRevenue IS NOT NULL
  AND totals.transactions IS NOT NULL
GROUP BY month;
```
| Month  | Avg_total_transactions_per_user |
|--------|---------------------------------|
| 201707 | 43.86                           |


Key takeaways:
- **Could be Frequency Outlier:** An average of 43.86 transactions per user is an extraordinary baseline. This indicates that the "average" customer is purchasing ~1.4 times every day.


### Query 7. Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017
```sql
WITH CUSTOMER AS(
SELECT DISTINCT fullVisitorId, product.productRevenue
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product 
where v2ProductName="YouTube Men's Vintage Henley" 
AND product.productRevenue is not null 
),

PRODUCT AS(
SELECT fullVisitorId, v2ProductName, productQuantity, product.productRevenue
FROM 
  `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
WHERE product.productRevenue is not null 
)

SELECT 
  PRODUCT.v2ProductName, 
  SUM(PRODUCT.productQuantity) AS quantity
FROM  CUSTOMER
INNER JOIN  PRODUCT
ON PRODUCT.fullVisitorId=CUSTOMER.fullVisitorId
WHERE 
  PRODUCT.v2ProductName NOT LIKE "YouTube Men's Vintage Henley" 
AND 
  product.productRevenue is not null 
GROUP BY PRODUCT.v2ProductName
ORDER BY SUM(PRODUCT.productQuantity) DESC
```
| other_purchased_products                                 | quantity |
|----------------------------------------------------------|----------|
| Google Sunglasses                                        | 20       |
| Google Womens Vintage Hero Tee Black                     | 7        |
| SPF-15 Slim & Slender Lip Balm                           | 6        |
| Google Womens Short Sleeve Hero Tee Red Heather          | 4        |
| YouTube Mens Fleece Hoodie Black                         | 3        |
| Google Mens Short Sleeve Badge Tee Charcoal              | 3        |
| YouTube Twill Cap                                        | 2        |
| Red Shine 15 oz Mug                                      | 2        |


Overall, the data provides valuable insights into customer preferences, product popularity, and potential areas for marketing and merchandising strategies. Further analysis of historical data and integration with customer demographics could provide a more comprehensive understanding of these trends.


### Query 8. Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.
```sql
with product_data as(
select
    format_date('%Y%m', parse_date('%Y%m%d',date)) as month,
    count(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) as num_product_view,
    count(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) as num_add_to_cart,
    count(CASE WHEN eCommerceAction.action_type = '6' THEN product.v2ProductName END) as num_purchase
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
from product_data
```
| month  | num_product_view | num_add_to_cart | num_purchase | add_to_cart_rate | purchase_rate |
|--------|------------------|-----------------|--------------|------------------|---------------|
| 201701 | 25787            | 7342            | 4328         | 28.47            | 16.78         |
| 201702 | 21489            | 7360            | 4141         | 34.25            | 19.27         |
| 201703 | 23549            | 8782            | 6018         | 37.29            | 25.56         |

Overall, the number of product views from January 2017 to March 2017 increased gradually. The add-to-cart rate and purchase rate also increased over the same period, indicating improved user engagement and conversion.

However, the add-to-cart rate and purchase rate are notably higher in March 2017, suggesting potential improvements in the website's user experience or marketing efforts.

## 4. Recommendations
- **Fix the "Mobile Gap":** Your 11% higher bounce rate on mobile Facebook (64% vs 53%) suggests a major UX or speed issue. Optimize landing pages for mobile-first loading to stop losing these users.
- **Facilitate the "Deep Dive":** Since purchasers view 6x more pages (25 vs 4) than non-purchasers, add side-by-side comparison tools and "recently viewed" carousels to help users research without friction.
- **Retarget Social "Window Shoppers":** Facebook and YouTube drive high traffic but almost zero revenue. Shift social spend from broad awareness to retargeting ads that show specific product reviews to users who already visited.
- **Boost Purchase Frequency:** Your current baseline is 1.11 transactions per user. Launch a loyalty or "refill" email program to nudge this closer to 1.5, capitalizing on your strong Direct traffic ($195k/mo).



