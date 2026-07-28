# E-Commerce Sales Funnel Analysis


## Project Overview

This project analyzes user behavior across an ecommerce sales funnel using Google BigQuery SQL. The analysis focuses on the first 30 days of user activity and measures conversion rates, traffic source performance, customer journey timing, and revenue metrics to identify opportunities for improving ecommerce performance.


## Business Questions Answered 

In this project I tried to answer the following questions 

1.How many users reach each stage of the sales funnel?

2.Where do users drop off before purchasing?

3.Which marketing channel converts best?

4.How long does it take customers to complete a purchase?

5.What is the average order value and revenue per customer?

## Dataset 

I've uploaded the user_events.csv file to this repository. It is an e-commerce dataset containing 9,381 rows of user event data, including page views, add-to-cart actions, checkout events, payment information, purchases, traffic sources (such as organic, email, and paid ads), timestamps, and user IDs.

The structure of the dataset is as follows:
<img width="1405" height="456" alt="image" src="https://github.com/user-attachments/assets/2c1ab22f-3c69-4164-8011-ec3b40d029af" />

## Tools

Google BigQuery |
SQL |
Github

## Analysis

**1.Funnel Analysis **

I'm trying to identify the number of users at each event type by creating a funnel analysis query. The goal is to determine how many users progress through each stage of the funnel. For this analysis, I'm only considering data from the first 30 days to calculate the number of users at each stage of the funnel.

```sql
WITH funnel_stages AS 
(

SELECT 

COUNT(DISTINCT CASE WHEN event_type ='page_view'   THEN user_id  END)  as  stage1_page_view,
COUNT(DISTINCT CASE WHEN event_type ='add_to_cart' THEN user_id  END)  as  stage2_add_to_cart,
COUNT(DISTINCT CASE WHEN event_type ='checkout_start' THEN user_id  END)  as  stage3_checkout_start,
COUNT(DISTINCT CASE WHEN event_type ='payment_info' THEN user_id  END)  as  stage4_payment_info,
COUNT(DISTINCT CASE WHEN event_type ='purchase' THEN user_id  END)  as  stage5_purchase_info

FROM `proven-entropy-339205.User_Events.user_events` 


## Min date is Dec 30 2025 , my goal is to analyze first 30 days data which will bring me to Jan 29 , 2026. ( date_sub - will subtract days , date_add will add days based on the interval specified)


WHERE event_date >= (SELECT MIN(event_date) FROM `proven-entropy-339205.User_Events.user_events`)  AND event_date <= TIMESTAMP(DATE_ADD((SELECT MIN(event_date) FROM `proven-entropy-339205.User_Events.user_events`),INTERVAL 30 DAY ))

)
```
**Insights **: We can observe that the number of users decreases at each stage of the funnel. In other words, not all users who view pages progress to the final purchase stage.
<img width="1483" height="152" alt="image" src="https://github.com/user-attachments/assets/1377ecfe-ad13-4489-b404-e13e647430f8" />

---

**2.Conversion Analysis**

Analyzing Conversion Rates through the funnel , basically we are trying to identify the ratios of one stage to the other 
**A conversion rate tells us:** "Out of the users who reached the previous stage, what percentage successfully moved to the next stage?

```sql
SELECT 
ROUND(stage2_add_to_cart*100.0/stage1_page_view,2)  AS page_view_to_add_to_cart_rate,  ## Here we are checking out of all the users who viewed pages how many procceded  to add to cart 
ROUND(stage3_checkout_start*100.0/stage2_add_to_cart,2) AS add_to_cart_to_checkout_start_rate, ## out of all the users who added to cart how many started the check out 
ROUND(stage4_payment_info*100.0/stage3_checkout_start,2) AS checkout_start_to_payment_rate, ## out of all the users who started checkout proceeded to payment 
ROUND(stage5_purchase_info*100.0/stage4_payment_info,2) AS payment_to_purchase_rate, ## out of all the users who proceeded to payment who actually went ahead and purchased
ROUND(stage5_purchase_info*100.0/stage1_page_view,2) AS overall_purchase_rate   ## out of all the users who viewed the page how many actually purchased 
FROM funnel_stages ;
```
<img width="1485" height="149" alt="image" src="https://github.com/user-attachments/assets/50f57a16-bc8d-4ac8-a051-d2e06a1abe97" />

**Insights:**

Based on the results of the above query, we are trying to understand why many users who started by viewing pages did not progress to the purchase stage, and why only **16.3%** of users completed a purchase within the first 30 days.

To investigate this drop-off, we can explore questions such as:

* Were users unable to find the products or information they were looking for?
* Was the **"Add to Cart"** button difficult to find or not prominently displayed?
* Were product prices discouraging users from completing their purchases?
* Was the checkout process too long or complicated?
* Were there any issues with payment or shipping that caused users to abandon their purchases?

Answering these questions can help identify friction points in the user journey and reveal opportunities to improve the overall conversion rate.
