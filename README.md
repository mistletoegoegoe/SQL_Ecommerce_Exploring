# [SQL-Big Query] Website Performance Analysis in eCommerce 
## 1. Business requirements

The project was conducted to assist **Sales and Marketing Manager** to have an outlook on the business situation and marketing efficiency by analysing performance of the website, in terms of **Revenue, Bounce rate, The patterns of user behaviour, Products, Customer journey, etc**. 


The dataset contains records about user sessions on a website collected from Google Analytics in 2017.


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


Bounce rate is defined as the percentage of visitors that leave a webpage without taking an action, such as clicking on a link, filling out a form, or making a purchase.

This table showed the overview of website traffic, in regarding to different source, and the metrics that helped to analyse the level of engagement in the users as well as their behaviour. Those factors that were examined: source, total_visits, total_no_of_bounces, bounce_rate.

#### 4.2.1 Sources with high number of visits

Based on the table, sources that had the highest total number of visits are Google, (direct), Youtube, analytics.google.com, Partners, m.facebook.com. The same subsequency occured in terms of number of bounce. For those sources, the bounce rates were respectively 51.56%, 43.27%, 66.73%, 53.96%, 52.35%, 64.28%. These top sources commonly had bounce rate ranging from around 40-65%. 

#### 4.2.2 Sources with low number of visits

On the other hand, suche.t-online.de had the lowest number of visits (only 1) but the highest percentage of users who leave without any futher action (100%). The similar situation happened with online.fullsail.edu, web.facebook.com, images.google.com.au, mx.search.yahoo.com, it.pinterest.com, es.search.yahoo.com, news.ycombinator.com, google.bg, web.mail.comcast.net, kik.com, gophergala.com, kidrex.org, malaysia.search.yahoo.com, google.es. This might indicated that either these websites did not contain the content they were looking for or the landing pages were not appealing enough for them to stay and explore further. 

#### 4.2.3 Sources with low bounce rate

Reddit , Mail.google.com, Google.ru and Hangouts.google.com were 4 sources with the lowest bounce rate, ranging from 20-30%. However, besides Reddit and Mail.google.com, the other two sources only had a very modest number of visits (5 each). Thus, it might be biased if any conclusions would be drawn from these two sources. For Reddit and Mail.google.com, the low bounce rate might indicate that the content on these platforms were interesting to some extent to their users, or the marketing campaigns that were implemented via Gmail effectively targeted the potential customers. 

#### 4.2.4 Social media performace

Facebook.com and M.facebook.com were 2 link that directed to Facebook platform. With total visits of 191 and 669, respectively, their bounce rate were 53.4% and 64.28%, which suggested average engagement from Facebook users. In addition, more than 64% of mobile facebook (m.facebook.com) might showed the potential issues with experience of  mobile Facebook users or content relevancy.

#### 4.2.5 Conclusion

However, while bounce rate could tell some valuable insights of user behaviour and their engagement in the content on website, it was also necessary to assess thoroughly the context of those sources and user action, to draw a comprehensive conclusion about the meaning of bounce rate. 

### 4.3. Revenue by traffic source by week, by month in June 2017
```
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
| week      | 201722 | (direct)          | 6,884.90  |
| week      | 201726 | google            | 5,330.57  |
| week      | 201726 | dfa               | 3,704.74  |
| month     | 201706 | mail.google.com   | 2,563.13  |
| week      | 201724 | mail.google.com   | 2,486.86  |
| week      | 201724 | dfa               | 2,341.56  |
| week      | 201722 | google            | 2,119.39  |
| week      | 201722 | dfa               | 1,670.65  |
| week      | 201723 | dfa               | 1,124.28  |
| week      | 201723 | google            | 1,083.95  |
| week      | 201725 | google            | 1,006.10  |
| week      | 201723 | search.myway.com  | 105.94    |
| month     | 201706 | search.myway.com  | 105.94    |
| month     | 201706 | groups.google.com | 101.96    |
| week      | 201725 | mail.google.com   | 76.27     |
| week      | 201723 | chat.google.com   | 74.03     |
| month     | 201706 | chat.google.com   | 74.03     |
| month     | 201706 | dealspotr.com     | 72.95     |
| week      | 201724 | dealspotr.com     | 72.95     |
| month     | 201706 | mail.aol.com      | 64.85     |
| week      | 201725 | mail.aol.com      | 64.85     |
| week      | 201726 | groups.google.com | 63.37     |
| week      | 201725 | phandroid.com     | 52.95     |
| month     | 201706 | phandroid.com     | 52.95     |
| month     | 201706 | sites.google.com  | 39.17     |
| week      | 201725 | groups.google.com | 38.59     |
| week      | 201725 | sites.google.com  | 25.19     |
| week      | 201725 | google.com        | 23.99     |
| month     | 201706 | google.com        | 23.99     |
| month     | 201706 | yahoo             | 20.39     |
| week      | 201726 | yahoo             | 20.39     |
| week      | 201723 | youtube.com       | 16.99     |
| month     | 201706 | youtube.com       | 16.99     |
| week      | 201724 | bing              | 13.98     |
| month     | 201706 | bing              | 13.98     |
| week      | 201722 | sites.google.com  | 13.98     |
| week      | 201724 | l.facebook.com    | 12.48     |
| month     | 201706 | l.facebook.com    | 12.48     |

The above table showed the total revenue generated by time and sources. Each row corresponds to a specific time period (week or month) and provides information about the revenue generated from different sources during that time period. Insights could be obtained were: 

(Direct) source was the most significant revenue contributor, accounting for $194,463.24 in June 2017 alone. The highest weekly revenue from direct traffic was $30,883.91 during week 24 (June 12-18, 2017), indicating a strong base of users who visit the website directly. This suggests high brand loyalty or effective offline marketing strategies. The consistent revenue from direct traffic across multiple weeks highlights the importance of maintaining and enhancing these direct user relationships.

Google is the second-highest source of revenue ($37,514.36 in June). This indicated the effectiveness of SEO tool, and potential paid search campaigns. This underscores the necessity of search engines to drive traffic and revenue. DFA (DoubleClick for Advertisers) also plays a vital role, contributing $17,682.46 for the month, proven the effectiveness of display advertising in pulling visitors.

Email marketing also appears to be an important channel, with the next high revenue generation. This showed that the email marketing campaigns had positively impacted on the targeted audience and led to conversions. 

m.facebook.com and youtube.com, although they had a significant traffic, they brang a very small amount of revenue. That could be explained by the high bounce rate that were figured out above. This implied a need for better targeted content or improve user experience to keep them in the website longers, which might increase the chance of conversions.

In conclusion, the table primarily demonstrated the importance of marketing activities on digital platforms, including engaging content, advertising, email marketing, search engine optimisation to maximise revenue, conversions and user visits. 

### 4.4. Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
```
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

#### Pageviews of purchaser and non-purchaser
There was a significant difference in the number of pageviews that purchasers and non-purchasers had made. On average, users who made transactions tended to view around 25 pages, equals 6 times of this figure of non-purchasers (about 4 pages). 
#### User engagement
The higher average pageviews of purchasers suggested that the more engaged to website content, the more likely users made transactions. 
#### User bahaviour
The users who had intent to buy in advance, tended to spend more time in researching product information, browsing product reviews, details, or compared with other options before making payment. That led to the higher number of pageviews. 
#### Recommendations
The website should be optimised to showcase the relevant information such as product details, reviews, other similar products, etc, in order to save time for users and enhance their experience. 
### 4.5. Average number of transactions per user that made a purchase in July 2017
```
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

The table shows the average total transactions per user in July. This data suggests that, during July 2017, the typical user conducted about 1.11 transactions on average. This could be useful for understanding user behavior, tracking user engagement with your platform, or evaluating the effectiveness of marketing campaigns or promotions during that specific month.

### 4.6. Average amount of money spent per session. Only include purchaser data in July 2017
```

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

The average total transactions per user for July 2017 is 43.86. This suggests that, on average, each user conducted approximately 44 transactions during that month. This could be an important metric for businesses to measure user engagement and activity.

### 4.7. Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017
```
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
| Google Doodle Decal                                      | 2        |
| Recycled Mouse Pad                                       | 2        |
| Google Mens Short Sleeve Hero Tee Charcoal               | 2        |
| Android Womens Fleece Hoodie                             | 2        |
| 22 oz YouTube Bottle Infuser                             | 2        |
| Android Mens Vintage Henley                              | 2        |
| Crunch Noise Dog Toy                                     | 2        |
| Android Wool Heather Cap Heather/Black                   | 2        |
| Google Mens Vintage Badge Tee Black                      | 1        |
| Google Twill Cap                                         | 1        |
| Google Mens Long & Lean Tee Grey                         | 1        |
| Google Mens Long & Lean Tee Charcoal                     | 1        |
| Google Laptop and Cell Phone Stickers                    | 1        |
| Google Mens Bike Short Sleeve Tee Charcoal               | 1        |
| Google 5-Panel Cap                                       | 1        |
| Google Toddler Short Sleeve T-shirt Grey                 | 1        |
| Android Sticker Sheet Ultra Removable                    | 1        |
| YouTube Custom Decals                                    | 1        |
| Four Color Retractable Pen                               | 1        |
| Google Mens Long Sleeve Raglan Ocean Blue                | 1        |
| Google Mens Vintage Badge Tee White                      | 1        |
| Google Mens 100% Cotton Short Sleeve Hero Tee Red        | 1        |
| Android Mens Vintage Tank                                | 1        |
| Google Mens Performance Full Zip Jacket Black            | 1        |
| 26 oz Double Wall Insulated Bottle                       | 1        |
| Google Mens Zip Hoodie                                   | 1        |
| YouTube Womens Short Sleeve Hero Tee Charcoal            | 1        |
| Google Mens Pullover Hoodie Grey                         | 1        |
| YouTube Mens Short Sleeve Hero Tee White                 | 1        |
| Android Mens Short Sleeve Hero Tee White                 | 1        |
| Android Mens Pep Rally Short Sleeve Tee Navy             | 1        |
| YouTube Mens Short Sleeve Hero Tee Black                 | 1        |
| Google Slim Utility Travel Bag                           | 1        |
| Android BTTF Moonshot Graphic Tee                        | 1        |
| Google Mens Airflow 1/4 Zip Pullover Black               | 1        |
| Google Womens Long Sleeve Tee Lavender                   | 1        |
| 8 pc Android Sticker Sheet                               | 1        |
| YouTube Hard Cover Journal                               | 1        |
| Android Mens Short Sleeve Hero Tee Heather               | 1        |
| YouTube Womens Short Sleeve Tri-blend Badge Tee Charcoal | 1        |
| Google Mens Performance 1/4 Zip Pullover Heather/Black   | 1        |
| YouTube Mens Long & Lean Tee Charcoal                    | 1        |

Overall, the data provides valuable insights into customer preferences, product popularity, and potential areas for marketing and merchandising strategies. Further analysis of historical data and integration with customer demographics could provide a more comprehensive understanding of these trends.
### 4.8. Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.
```
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

## 5. Gains from the project
- Conducted this project was a huge opportunity to learn about marketing industry, especially customers' behaviour on digital platforms. That could be gained by analysing metrics such as bounce rate, revenue, transactions per visit, etc. 
- Gained insights about marketing channels, how those channels drove traffic and pull customers, and the contribution of sources in total revenue.
- Put foward some recommendations to improved website content and enhance user experience. 


