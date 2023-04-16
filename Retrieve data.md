 ## 📄 Retrieve reports on transaction scenarios



- Retrieve a report that includes the following information: customer_id, transaction_id, scenario_id, transaction_type, sub_category, category. These transactions must meet the following conditions: 
    - Were created in Jan 2019 
    - Transaction type is not payment
````sql
select customer_id
    , transaction_id
    , fact_2019.scenario_id
    , sub_category
    , category
    , transaction_time
from fact_transaction_2019 as fact_2019
join [dbo].[dim_scenario] as scena
on fact_2019.scenario_id = scena.scenario_id
where transaction_time between '2019-01-01' and '2019-02-01'
and transaction_type !='Payment'
order by transaction_time desc
````
Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232201807-1f636270-2896-4dd7-a69d-ba1e73414a5f.png">
</picture>
 
 
- Retrieve a report that includes the following information: customer_id, transaction_id, scenario_id, transaction_type, category, payment_method. These transactions must meet the following conditions: 
    - Were created from Jan to June 2019
    - Had category type is shopping
    - Were paid by Bank account

````sql
SELECT  fact_19.customer_id
    ,fact_19.transaction_id
    ,fact_19.scenario_id
    ,se.transaction_type
    ,se.category
    ,pay_cha.payment_method
FROM [dbo].[dim_scenario] AS se
INNER JOIN [dbo].[fact_transaction_2019] AS fact_19
ON se.scenario_id = fact_19.scenario_id
INNER JOIN [dbo].[dim_payment_channel] AS pay_cha
ON fact_19.payment_channel_id = pay_cha.payment_channel_id
WHERE (MONTH(transaction_time) BETWEEN 1 AND 6)
    AND se.category LIKE 'Shopping'
    AND pay_cha.payment_method LIKE 'Bank account'
````


Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232202084-c5d79506-710a-45ab-a217-5b172d6ed781.png">
</picture>

-  Retrieve a report that includes the following information: customer_id, transaction_id, scenario_id, payment_method and payment_platform. These transactions must meet the following conditions: 
    - Were created in Jan 2019 and Jan 2020 
    - Had payment platform is android

````sql
SELECT  fact_19.customer_id
            , fact_19.transaction_id
            ,fact_19.scenario_id
            , pay_cha.payment_method
            , pla.payment_platform
FROM [dbo].[fact_transaction_2019] AS fact_19
INNER JOIN [dbo].[dim_payment_channel] AS pay_cha
ON fact_19.payment_channel_id = pay_cha.payment_channel_id
INNER JOIN [dbo].[dim_platform] as pla
ON fact_19.platform_id = pla.platform_id
WHERE pla.payment_platform LIKE 'android'
    AND MONTH(transaction_time) = 1
UNION 
SELECT  fact_20.customer_id
            , fact_20.transaction_id
            ,fact_20.scenario_id
            , pay_cha.payment_method
            , pla.payment_platform
FROM  [dbo].[fact_transaction_2020] AS fact_20
INNER JOIN [dbo].[dim_payment_channel] AS pay_cha
ON fact_20.payment_channel_id = pay_cha.payment_channel_id
INNER JOIN [dbo].[dim_platform] as pla
ON fact_20.platform_id = pla.platform_id
WHERE pla.payment_platform LIKE 'android'
    AND MONTH(transaction_time) = 1
````


Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232202375-dc513be9-0ff6-4b5e-a842-e638439d3016.png">
 </picture>

- Retrieve a report that includes the following information: customer_id, transaction_id,scenario_id, payment_method and payment_platform. 
These transactions must meet the following conditions:
    - Include all transactions of the customer group created in January 2019 (Group A) and additional transactions of this customers (Group A) continue to make transactions in January 2020.
    - Payment platform is iOS
 
 ````sql

SELECT customer_id
        ,transaction_id
        ,scenario_id
        ,payment_method
        ,payment_platform
FROM [dbo].[fact_transaction_2019] AS fact_19
join [dbo].[dim_platform] as platform
on fact_19.platform_id = platform.platform_id
join [dbo].[dim_payment_channel] as channel
on fact_19.payment_channel_id = channel.payment_channel_id
WHERE MONTH(transaction_time) = 1
and payment_platform ='ios'

UNION

SELECT  fact_20.customer_id
        , fact_20.transaction_id
        , fact_20.scenario_id
        , channel.payment_method
        , platform.payment_platform
FROM [dbo].[fact_transaction_2020] AS fact_20
INNER JOIN [dbo].[dim_payment_channel] AS channel
ON fact_20.payment_channel_id = channel.payment_channel_id
INNER JOIN [dbo].[dim_platform] as platform
ON fact_20.platform_id = platform.platform_id
WHERE platform.payment_platform LIKE 'ios'
AND MONTH(transaction_time) = 1
AND customer_id in (
    SELECT Distinct   fact_19.customer_id
    FROM [dbo].[fact_transaction_2019] AS fact_19
    INNER JOIN [dbo].[dim_payment_channel] AS channel
    ON fact_19.payment_channel_id = channel.payment_channel_id
    INNER JOIN [dbo].[dim_platform] as platform
    ON fact_19.platform_id = platform.platform_id
    WHERE platform.payment_platform LIKE 'ios'
    AND MONTH(transaction_time) = 1
    )

````


Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232202525-cc5ffcd7-4d5a-43f8-ba8f-3edfdab6bcf8.png">
</picture>

## 💰 Retrieve an overview report of payment types
 
 
Paytm has a wide variety of transaction types in its business. Your manager wants to know the contribution(by percentage) of each transaction type to total transactions. 
 
- Retrieve a report that includes the following information: transaction type, number of transaction and proportion of each type in total. 
These transactions must meet the following conditions: 
  - Were created in 2019 
  - Were paid successfully
````sql
WITH type_table AS(
        SELECT  fact_19.*
        , transaction_type
        FROM [dbo].[fact_transaction_2019] AS fact_19
        JOIN [dbo].[dim_status] AS stat
        ON fact_19.status_id = stat.status_id
        JOIN [dbo].[dim_scenario] AS scena
        ON fact_19.scenario_id = scena.scenario_id
        WHERE status_description = 'Success'
)
, total_table as (
SELECT  transaction_type
        , count(transaction_id) as number_trans
        , (select count(transaction_id ) from type_table)  as total_trans 
from  type_table
group by transaction_type
)

select top 5 
    * 
    , FORMAT(number_trans*1.0/ total_trans,'p') as pct
from total_table
order by number_trans desc
````
Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232202795-7bb3c9e2-2d7f-4b51-91aa-b4058fdeee9c.png">
</picture>


- Retrieve a more detailed report with following information: 
transaction type, category, number of transaction and proportion of each category in the total of that transaction type. 
These transactions must meet the following conditions: 
  - Were created in 2019 
  - Were paid successfully
 
 ````sql
WITH type_table AS(
        SELECT  fact_19.*
        , transaction_type
        , category
        FROM [dbo].[fact_transaction_2019] AS fact_19
        JOIN [dbo].[dim_status] AS stat
        ON fact_19.status_id = stat.status_id
        JOIN [dbo].[dim_scenario] AS scena
        ON fact_19.scenario_id = scena.scenario_id
        WHERE status_description = 'Success'
)
, category_table AS ( 
  SELECT category
        , transaction_type
        , COUNT (transaction_id) AS number_trans_category
FROM type_table
     GROUP BY category, transaction_type
) 
, count_type AS(
    select transaction_type
        , count(transaction_id) as number_trans_type
    from type_table
    group by transaction_type
)

select category_table.*
    , number_trans_type
    , FORMAT(number_trans_category*1.0/number_trans_type,'p') as pct
from category_table
join count_type 
on category_table.transaction_type = count_type.transaction_type
where number_trans_type is not null 
and number_trans_category is not NULL
order by number_trans_category*1.0/ number_trans_type DESC

````

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232202901-eb0e0ab9-fc88-4a8f-8d73-dc273e119ade.png">
</picture>
 
 

## 🙎 Retrieve an overview report of customer’s payment behaviors

Paytm has acquired a lot of customers. 
- Retrieve a report that includes the following information: the number of transactions, the number of payment scenarios, the number of transaction types,  the number of payment category and the total of charged amount of each customer.
  - 	Were created in 2019
  - Had status description is successful
  - Had transaction type is payment
  - Only show Top 10 highest customers by the number of transactions

````sql

WITH success_table AS (
SELECT  customer_id 
        , fact_19.transaction_id
        , fact_19.scenario_id
        , fact_19.status_id
        , category, fact_19.charged_amount
FROM [dbo].[fact_transaction_2019] AS fact_19
JOIN [dbo].[dim_status] AS sta
ON fact_19.status_id = sta.status_id
JOIN [dbo].[dim_scenario] AS scena
ON fact_19.scenario_id = scena.scenario_id 
WHERE transaction_type = 'Payment'
        AND status_description = 'Success')

SELECT top 10 customer_id 
    , COUNT (transaction_id) AS number_trans
    ,COUNT (DISTINCT scenario_id) AS number_scenarios
    ,COUNT (DISTINCT category) AS number_categories
    ,SUM (charged_amount) AS total_amount 
FROM success_table
GROUP BY customer_id   
ORDER by number_trans DESC

````
Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232203239-688c6ab3-9807-4d05-b9c2-ce22a2aec1fc.png">
</picture>

- After looking at the above metrics of customer’s payment behaviors,  we want to analyze the distribution of each metric.  Before calculating and plotting the distribution to check the frequency of values in each metric, we need to group the observations into range.
Based on the result of 2.1, retrieve a report that includes the following columns: metric, minimum value, maximum value and average value of these metrics: 
   - The total charged amount
   - The number of transactions
   - The number of payment scenarios
   - The number of payment categories

````sql
WITH summary_table AS (
SELECT customer_id
    , COUNT(transaction_id) AS number_trans
    , COUNT(DISTINCT fact_19.scenario_id) AS number_scenarios
    , COUNT(DISTINCT scena.category) AS number_categories
    , SUM(charged_amount) AS total_amount
FROM fact_transaction_2019 AS fact_19
LEFT JOIN dim_scenario AS scena 
        ON fact_19.scenario_id = scena.scenario_id
LEFT JOIN dim_status AS sta 
        ON fact_19.status_id = sta.status_id 
WHERE status_description = 'success'
    AND transaction_type = 'payment'
GROUP BY customer_id
)
SELECT 'The number of transaction' AS metric 
    , MIN(number_trans) AS min_value
    , MAX(number_trans) AS max_value
    , AVG(number_trans) AS avg_value
FROM summary_table
UNION 
SELECT 'The number of scenarios' AS metric
    , MIN(number_scenarios) AS min_value
    , MAX(number_scenarios) AS max_value
    , AVG(number_scenarios) AS avg_value
FROM summary_table
UNION 
SELECT 'The number of categories' AS metric
    , MIN(number_categories) AS min_value
    , MAX(number_categories) AS max_value
    , AVG(number_categories) AS avg_value
FROM summary_table
UNION 
SELECT 'The total charged amount' AS metric
    , MIN(total_amount) AS min_value
    , MAX(total_amount) AS max_value
    , AVG(1.0*total_amount) AS avg_value
FROM summary_table

-- Metric 1: The number of payment categories */ 
WITH summary_table AS (
SELECT customer_id
    , COUNT(DISTINCT scena.category) AS number_categories
FROM fact_transaction_2019 AS fact_19
LEFT JOIN dim_scenario AS scena 
        ON fact_19.scenario_id = scena.scenario_id
LEFT JOIN dim_status AS sta 
        ON fact_19.status_id = sta.status_id 
WHERE status_description = 'success'
    AND transaction_type = 'payment'
GROUP BY customer_id -- 
)
SELECT number_categories
    , COUNT(customer_id) AS number_customers
FROM summary_table
GROUP BY number_categories 
ORDER BY number_categories

-- Metric 2: The number of payment scenarios
WITH summary_table AS (
SELECT customer_id
    , COUNT(DISTINCT fact_19.scenario_id) AS number_scenarios
FROM fact_transaction_2019 AS fact_19
LEFT JOIN dim_scenario AS scena 
        ON fact_19.scenario_id = scena.scenario_id
LEFT JOIN dim_status AS sta 
        ON fact_19.status_id = sta.status_id 
WHERE status_description = 'success'
    AND transaction_type = 'payment'
GROUP BY customer_id
)
SELECT number_scenarios
    , COUNT(customer_id) AS number_customers
FROM summary_table
GROUP BY number_scenarios 
ORDER BY number_scenarios




--Metric 3: The total charged amount 

WITH summary_table AS (
SELECT customer_id
    , SUM(charged_amount) AS total_amount
    , CASE
        WHEN SUM(charged_amount) < 1000000 THEN '0-01M'
        WHEN SUM(charged_amount) >= 1000000 AND SUM(charged_amount) < 2000000 THEN '01M-02M'
        WHEN SUM(charged_amount) >= 2000000 AND SUM(charged_amount) < 3000000 THEN '02M-03M'
        WHEN SUM(charged_amount) >= 3000000 AND SUM(charged_amount) < 4000000 THEN '03M-04M'
        WHEN SUM(charged_amount) >= 4000000 AND SUM(charged_amount) < 5000000 THEN '04M-05M'
        WHEN SUM(charged_amount) >= 5000000 AND SUM(charged_amount) < 6000000 THEN '05M-06M'
        WHEN SUM(charged_amount) >= 6000000 AND SUM(charged_amount) < 7000000 THEN '06M-07M'
        WHEN SUM(charged_amount) >= 7000000 AND SUM(charged_amount) < 8000000 THEN '07M-08M'
        WHEN SUM(charged_amount) >= 8000000 AND SUM(charged_amount) < 9000000 THEN '08M-09M'
        WHEN SUM(charged_amount) >= 9000000 AND SUM(charged_amount) < 10000000 THEN '09M-10M'
        WHEN SUM(charged_amount) >= 10000000 THEN 'more > 10M'
        END AS charged_amount_range
FROM fact_transaction_2019 AS fact_19
LEFT JOIN dim_scenario AS scena
        ON fact_19.scenario_id = scena.scenario_id
LEFT JOIN dim_status AS sta 
        ON fact_19.status_id = sta.status_id 
WHERE status_description = 'success'
    AND transaction_type = 'payment'
GROUP BY customer_id
)
SELECT charged_amount_range
    , COUNT(customer_id) AS number_customers
FROM summary_table
GROUP BY charged_amount_range 
ORDER BY charged_amount_range
````
Answer:

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232203354-68c5a394-6519-4fe8-83b1-dff1d58a378c.png">
</picture>

  - The number of payment categories
 <picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232203402-70c81ddb-c664-4959-b35f-89af2f738a74.png">
</picture>

  - The number of payment scenarios
 <picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232203402-70c81ddb-c664-4959-b35f-89af2f738a74.png">
</picture>

  - The total charged amount 

  <picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232204649-aff9f09a-5583-4525-ab78-3a5987140325.png">
</picture>



## ⏳ Time Series Analysis

- Trending the Data
You need to analyze the trend of payment transactions of Billing category from 2019 to 2020.
First, let’s show the trend of the number of successful transactions by month.


 ````sql 
 WITH fact_table AS (
    SELECT fact_19.*, category
    FROM fact_transaction_2019 fact_19 -- 400k 
    JOIN dim_scenario sce 
    ON fact_19.scenario_id = sce.scenario_id
    WHERE status_id = 1 AND category = 'Billing' 
UNION
    SELECT fact_20.*, category
    FROM fact_transaction_2020 fact_20 -- 800k 
    JOIN dim_scenario sce 
    ON fact_20.scenario_id = sce.scenario_id
    WHERE status_id = 1 AND category = 'Billing' 
)
SELECT Year(transaction_time) AS year, Month(transaction_time) AS month
    , COUNT(transaction_id) AS number_trans
FROM fact_table
GROUP BY Year(transaction_time), Month(transaction_time)
ORDER BY year, month
```` 
Answer:

<picture> 
 <img src="https://user-images.githubusercontent.com/129776645/232309158-89560b8e-399c-4787-abe3-15217bd2053b.png"> 
</picture>

- You know that there are many sub-categories of Billing group. 
After reviewing the above result, you should break down the trend into each sub-categories
````sql 
WITH month_table AS ( 
   SELECT transaction_id, customer_id,category,sub_category,  transaction_time, charged_amount
       , MONTH (transaction_time) AS [month]
       , YEAR (transaction_time) AS [year]
    FROM( SELECT * FROM fact_transaction_2019
           UNION
           SELECT * FROM fact_transaction_2020 ) AS fact_table
   JOIN dim_scenario AS scena
   ON fact_table.scenario_id = scena.scenario_id
   JOIN  [dbo].[dim_status] AS sta
    ON fact_table.status_id = sta.status_id
WHERE sta.status_description = 'Success'
    AND scena.category = 'Billing'
)
SELECT DISTINCT [year], [month], sub_category
                , COUNT (transaction_id) OVER 
                ( PARTITION BY [month], category, sub_category,[year]  ) AS number_trans
FROM month_table
ORDER BY  [year]
```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309291-7cc4170e-ebdf-45e5-9c80-73379e2ada9c.png"> 
</picture>

- Then modify the result as the following table: 
Only select the sub-categories belong to list (Electricity, Internet and Water)


````sql 
WITH fact_table AS ( 
    SELECT *
    FROM fact_transaction_2019 
    UNION 
    SELECT *
    FROM fact_transaction_2020 )  
, month_table AS (
    SELECT 
        Year(transaction_time) AS year, Month(transaction_time) AS month
        , sub_category
        , COUNT(transaction_id) AS number_trans
    FROM fact_table  
    JOIN dim_scenario AS sce ON fact_table.scenario_id = sce.scenario_id  
    WHERE status_id = 1 AND category = 'Billing' 
    GROUP BY Year(transaction_time), Month(transaction_time), sub_category
)
SELECT  year, month 
    , SUM ( CASE WHEN sub_category = 'Electricity' THEN number_trans ELSE 0  END ) AS electricity_trans
    , SUM ( CASE WHEN sub_category = 'Internet' THEN number_trans ELSE 0  END ) AS internet_trans
    , SUM ( CASE WHEN sub_category = 'Water' THEN number_trans ELSE 0  END ) AS water_trans
FROM month_table
GROUP BY year, month 
ORDER BY year, month

```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309502-39fc6a31-870e-4f96-b9d5-f8364038654a.png"> 
</picture>

- Based on the previous query, you need to calculate the proportion of each sub-category 
 (Electricity, Internet and Water) in the total for each month
 
 ````sql 
 WITH fact_table AS (
    SELECT *
    FROM fact_transaction_2019 
    UNION 
    SELECT *
    FROM fact_transaction_2020 )
, month_table AS (
    SELECT 
        YEAR(transaction_time) year, MONTH(transaction_time) month
        , sub_category
        , COUNT(transaction_id) AS number_trans
    FROM fact_table 
    JOIN dim_scenario AS sce ON fact_table.scenario_id = sce.scenario_id
    WHERE status_id = 1 AND category = 'Billing'
    GROUP BY YEAR(transaction_time), MONTH(transaction_time), sub_category
)
, pivot_month AS (
    SELECT Year 
        , month 
        , SUM( CASE WHEN sub_category = 'Electricity' THEN number_trans ELSE 0 END ) AS electricity_trans
        , SUM( CASE WHEN sub_category = 'Internet' THEN number_trans ELSE 0 END ) AS internet_trans
        , SUM( CASE WHEN sub_category = 'Water' THEN number_trans ELSE 0 END ) AS water_trans
    FROM month_table
    GROUP BY year, month
)
, total_month AS ( 
    SELECT * 
    , electricity_trans + internet_trans + water_trans AS total_trans_month
FROM pivot_month
)
SELECT *
    , FORMAT(1.0*electricity_trans/total_trans_month, 'p') AS elec_pct
    , FORMAT(1.0*internet_trans/total_trans_month, 'p') AS iternet_pct
    , FORMAT(1.0*water_trans/total_trans_month, 'p') AS water_pct
FROM total_month
ORDER BY year, month 

```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309566-26b42d1b-2473-4451-81e5-c505b67a5bc3.png"> 
</picture>

- Select only these sub-categories in the list (Electricity, Internet and Water), you need to calculate the number of successful paying customers for each month (from 2019 to 2020). 
Then find the percentage change from the first month (Jan 2019) for each subsequent month.

````sql 
WITH fact_table AS (
    SELECT * FROM fact_transaction_2019
    UNION 
    SELECT * FROM fact_transaction_2020
)
, customer_month AS (
    SELECT MONTH(transaction_time) month, YEAR(transaction_time) year
        , COUNT( DISTINCT customer_id ) AS number_customer -- đếm số lượng khách hàng 
    FROM fact_table
    JOIN dim_scenario AS scena ON fact_table.scenario_id = scena.scenario_id
    WHERE category = 'Billing' AND status_id = 1 AND sub_category IN ('Electricity', 'Internet',  'Water')
    GROUP BY MONTH(transaction_time), YEAR(transaction_time)
) 
, point_table AS (SELECT * 
    , (SELECT number_customer FROM customer_month WHERE year = 2019 AND month = 1) AS starting_point
FROM customer_month )

SELECT * 
    , FORMAT((number_customer*1.0 - starting_point*1.0)/starting_point,'p') AS different_pct
FROM point_table
```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309654-df3581a1-fc78-4c48-90e8-964eb14e0195.png"> 
</picture>

-  Select only these sub-categories in the list (Electricity, Internet and Water),
you need to calculate the number of successful paying customers for each week number from 2019 to 2020). 
Then get rolling annual paying users of this group.


````sql 
WITH fact_table AS (
   SELECT * FROM fact_transaction_2019
   UNION
   SELECT * FROM fact_transaction_2020
)
, week_user AS (
   SELECT YEAR(transaction_time) year, DATEPART(week, transaction_time) AS week_number
       , COUNT( DISTINCT customer_id ) AS number_customer
   FROM fact_table
   JOIN dim_scenario AS scena ON fact_table.scenario_id = scena.scenario_id
   WHERE category = 'Billing' AND status_id = 1 AND sub_category IN ('Electricity', 'Internet',  'Water')
   GROUP BY YEAR(transaction_time), DATEPART(week, transaction_time)

)
SELECT *
   , SUM (number_customer) OVER ( ORDER BY year, week_number  ) AS rolling_customers
FROM week_user

```` 
Answer:
<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309761-1cb50ac3-ec51-44ba-a5bd-b15987e3e9b7.png"> 
</picture>

- Then you calculate average number of customers of the last 4 weeks at each observation time
````sql 
WITH fact_table AS (
    SELECT * FROM fact_transaction_2019
    UNION 
    SELECT * FROM fact_transaction_2020
)
, week_user AS (
    SELECT YEAR(transaction_time) year, DATEPART(week, transaction_time) AS week_number
        , COUNT( DISTINCT customer_id ) AS number_customer
    FROM fact_table
    JOIN dim_scenario AS scena ON fact_table.scenario_id = scena.scenario_id
    WHERE category = 'Billing' AND status_id = 1 AND sub_category IN ('Electricity', 'Internet',  'Water')
    GROUP BY YEAR(transaction_time), DATEPART(week, transaction_time)

)
SELECT *
    , AVG(number_customer) OVER ( ORDER BY year, week_number ROWS BETWEEN 3 PRECEDING AND CURRENT ROW ) AS avg_last_4weeks 
FROM week_user
ORDER BY year, week_number 
```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309842-d0da4ac5-ffb5-409b-972b-e749e0668f02.png"> 
</picture>


## 💹 Cohort Analysis & User Segmentation


A. As you know that 'Telco Card' is the most product in the Telco group (accounting for more than 99% of the total). You want to evaluate the quality of user acquisition in Jan 2019 by the retention metric. 

- First, you need to know how many users are retained in each subsequent month from 
the first month (Jan 2019) they pay the successful transaction (only get data of 2019)

````sql 

WITH customer_list AS (
   SELECT DISTINCT customer_id
   FROM fact_transaction_2019 fact 
   JOIN dim_scenario sce ON fact.scenario_id = sce.scenario_id
   WHERE sub_category = 'Telco Card' AND status_id = 1 AND MONTH(transaction_time) = 1
)
, full_trans AS ( 
   SELECT fact.*
   FROM customer_list
   JOIN fact_transaction_2019 fact
       ON customer_list.customer_id = fact.customer_id
   JOIN dim_scenario sce
       ON fact.scenario_id = sce.scenario_id
   WHERE sub_category = 'Telco Card' AND status_id = 1
) 
SELECT MONTH(transaction_time) - 1 AS subsequence_month
   , COUNT( DISTINCT customer_id) AS retained_users
FROM full_trans
GROUP BY MONTH(transaction_time) - 1
ORDER BY subsequence_month

```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232309990-39f901ea-7e13-4a39-8aee-63da735dd6e9.png"> 
</picture>


B. You realize that the number of retained customers has decreased over time.

- Let’s calculate retention =  number of retained customers / total users of the first month


````sql 
WITH first_time_table AS (
   SELECT customer_id, transaction_id, transaction_time
       , MIN(transaction_time) OVER ( PARTITION BY customer_id ) AS first_time
       , DATEDIFF (month, MIN(transaction_time) OVER ( PARTITION BY customer_id ), transaction_time) AS subsequence_month
   FROM fact_transaction_2019 fact
   JOIN dim_scenario sce ON fact.scenario_id = sce.scenario_id
   WHERE sub_category = 'Telco Card' AND status_id = 1
)
, sub_month AS (
   SELECT subsequence_month
       , COUNT (DISTINCT customer_id) retained_users
   FROM first_time_table
   WHERE MONTH (first_time) = 1 
   GROUP BY subsequence_month
   
)
SELECT *
   , ( SELECT retained_users FROM sub_month WHERE subsequence_month = 0) AS original_1
   , FIRST_VALUE (retained_users) OVER (ORDER BY subsequence_month ASC ) AS original_2
   , FORMAT ( retained_users*1.0 / FIRST_VALUE (retained_users) OVER (ORDER BY subsequence_month ) , 'p') pct
FROM sub_month

```` 
Answer:

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232310054-e8756277-e7a2-444f-89a4-deb2a7a68cee.png"> 
</picture>

## 📌 Cohorts Derived from the Time Series Itself
Expend your previous query to calculate retention for multi attributes from the acquisition month (from Jan to December

````sql 
WITH first_time_table AS (
   SELECT customer_id, transaction_id, transaction_time
       , MIN(transaction_time) OVER ( PARTITION BY customer_id ) AS first_time
       , DATEDIFF (month, MIN(transaction_time) OVER ( PARTITION BY customer_id ), transaction_time) AS subsequence_month
   FROM fact_transaction_2019 fact
   JOIN dim_scenario sce ON fact.scenario_id = sce.scenario_id
   WHERE sub_category = 'Telco Card' AND status_id = 1
)
, sub_month AS (
   SELECT MONTH (first_time) AS acquisition_month
       , subsequence_month
       , COUNT (DISTINCT customer_id) retained_users
   FROM first_time_table
   GROUP BY MONTH (first_time) , subsequence_month
   -- ORDER BY acquisition_month, subsequence_month
)
SELECT *
   , FIRST_VALUE (retained_users) OVER (PARTITION BY acquisition_month ORDER BY subsequence_month ) AS original_users
   , FORMAT ( retained_users*1.0 / FIRST_VALUE (retained_users) OVER (PARTITION BY acquisition_month ORDER BY subsequence_month ) , 'p') pct
INTO #retention_table
FROM sub_month

SELECT * FROM #retention_table

SELECT acquisition_month, original_users
   , "0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11"
FROM (
   SELECT acquisition_month, subsequence_month, original_users, pct
   FROM #retention_table
   ) AS source_table
PIVOT (
   MIN (pct)
   FOR subsequence_month IN ("0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11")
) pivot_table
ORDER BY acquisition_month

```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232312252-bf7f322c-bf88-4fd1-befd-0f92573fe2c4.png"> 
</picture>

- The first step in building an RFM model is to assign Recency, Frequency and Monetary values to each customer. 
Let’s calculate these metrics for all successful paying customer of ‘Telco Card’ in 2019 and 2020: 
  - Recency: Difference between each customer's last payment date and '2020-12-31'
  - Frequency: Number of successful payment days of each customer
  - Monetary: Total charged amount of each customer

````sql 
-- Step 1: Tính RFM cho từng khách hàng
WITH fact_table AS (
   SELECT transaction_id, customer_id, scenario_id, charged_amount, transaction_time, status_id
   FROM fact_transaction_2019
   UNION
   SELECT transaction_id, customer_id, scenario_id, charged_amount, transaction_time, status_id
   FROM fact_transaction_2020
)
, rfm_table AS (
   SELECT customer_id
       , DATEDIFF ( day, MAX (transaction_time ), '2020-12-31') AS recency -- khoảng cách từ lần cuối so với ngày 31-12-2020
       , COUNT ( DISTINCT CONVERT (varchar(10), transaction_time) ) AS frequency -- đếm số ngày thanh toán thành công
       , SUM (charged_amount*1.0) AS monetary -- tính tổng tiền
   FROM fact_table
   LEFT JOIN dim_scenario scena
       ON fact_table.scenario_id = scena.scenario_id
   WHERE sub_category = 'Telco Card' AND status_id = 1
   GROUP BY customer_id
) -- b2: đánh thứ hạng r,f,m theo %
--> PERCENT_RANK(): đánh thứ hạng theo % (percentile)
, rank_table AS (
   SELECT *
       , PERCENT_RANK() OVER ( ORDER BY recency ASC ) AS r_rank
       , PERCENT_RANK() OVER ( ORDER BY frequency DESC ) AS f_rank
       , PERCENT_RANK() OVER ( ORDER BY monetary DESC ) AS m_rank
   FROM rfm_table
) -- chia tier
, tier_table AS (
   SELECT *
       , CASE WHEN r_rank > 0.75 THEN 4
           WHEN r_rank > 0.5 THEN 3
           WHEN r_rank > 0.25 THEN 2
           ELSE 1 END AS r_tier
       , CASE WHEN f_rank > 0.75 THEN 4
           WHEN f_rank > 0.5 THEN 3
           WHEN f_rank > 0.25 THEN 2
           ELSE 1 END AS f_tier
       , CASE WHEN m_rank > 0.75 THEN 4
           WHEN m_rank > 0.5 THEN 3
           WHEN m_rank > 0.25 THEN 2
           ELSE 1 END AS m_tier
   FROM rank_table
)
, score_table AS (
   SELECT customer_id, recency, frequency, r_rank, f_rank, m_rank
       , CONCAT ( r_tier, f_tier, m_tier) AS rfm_score
   FROM tier_table
)
, segment_table AS (
   SELECT *
       , CASE
       WHEN rfm_score  =  111 THEN 'Best Customers' -- KH tốt nhất
       WHEN rfm_score LIKE '[3-4][3-4][1-4]' THEN 'Lost Bad Customer' -- KH rời bỏ mà còn siêu tệ (F thấp)
       WHEN rfm_score LIKE '[3-4]2[1-4]' THEN 'Lost Customers' -- KH cũng rời bỏ nhưng có valued (F = 3,4,5)
       WHEN rfm_score LIKE  '21[1-4]' THEN 'Almost Lost' -- sắp lost những KH này
       WHEN rfm_score LIKE  '11[2-4]' THEN 'Loyal Customers'
       WHEN rfm_score LIKE  '[1-2][1-3]1' THEN 'Big Spenders' -- chi nhiều tiền
       WHEN rfm_score LIKE  '[1-2]4[1-4]' THEN 'New Customers' -- KH mới nên là giao dịch ít
       WHEN rfm_score LIKE  '[3-4]1[1-4]' THEN 'Hibernating' -- ngủ đông (trc đó từng rất là tốt )
       WHEN rfm_score LIKE  '[1-2][2-3][2-4]' THEN 'Potential Loyalists' -- có tiềm năng
       ELSE 'unknown'
       END segment_label
   FROM score_table
)
SELECT segment_label
   , COUNT ( customer_id ) AS number_customer
   , SUM ( COUNT ( customer_id ) ) OVER () AS total_customer
   , FORMAT ( COUNT ( customer_id )*1.0 / SUM ( COUNT ( customer_id ) ) OVER () , 'p') AS pct
FROM segment_table
GROUP BY segment_label
```` 
Answer: 

<picture>
 <img src="https://user-images.githubusercontent.com/129776645/232313344-04552ad6-70c8-467c-babf-bbb0ea7293fc.png"> 
</picture>











