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
