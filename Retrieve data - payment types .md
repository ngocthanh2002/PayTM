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
