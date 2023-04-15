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
 
