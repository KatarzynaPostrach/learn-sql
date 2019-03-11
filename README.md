# treść zadania poniżej kodu

 SELECT *
 FROM subscriptions
 LIMIT 10;
 
 SELECT MIN(subscription_end), MAX(subscription_end)
 FROM 
 subscriptions;
 
 SELECT MIN(subscription_start), MAX(subscription_start)
 FROM 
 subscriptions;
 
 --month table
 
WITH months AS
(SELECT
  '2017-01-01' as first_day,
  '2017-01-31' as last_day
UNION
SELECT
  '2017-02-01' as first_day,
  '2017-02-28' as last_day
UNION
SELECT
  '2017-03-01' as first_day,
  '2017-03-31' as last_day),

cross_join as 
(SELECT * 
FROM subscriptions CROSS JOIN months
),

status AS
(
SELECT *,
 	CASE
  WHEN 
 		(subscription_start < first_day)
  	AND 
 		((subscription_end > first_day)
    OR (subscription_end IS NULL)) AND
  	segment = 87
 	THEN 1
	ELSE 0
  END as is_active_87,
  
CASE
  WHEN 
 		(subscription_start < first_day)
  	AND 
 		((subscription_end > first_day)
    OR (subscription_end IS NULL)) AND
  	segment = 30
 	THEN 1
	ELSE 0
  END as is_active_30
 ,
  
CASE
 WHEN 
 		(subscription_start < first_day)
 	AND 
		(subscription_end between first_day AND last_day) AND segment = 87
	THEN 1
	ELSE 0
 END as is_canceled_87
  ,
  
CASE
 WHEN 
 (subscription_start < first_day)
  AND 
	(subscription_end between first_day AND last_day) AND segment = 30
THEN 1
	ELSE 0
END as is_canceled_30

FROM cross_join)

SELECT *
FROM status
LIMIT 100;

--funkcje agregujce

WITH months AS
(SELECT
  '2017-01-01' as first_day,
  '2017-01-31' as last_day
UNION
SELECT
  '2017-02-01' as first_day,
  '2017-02-28' as last_day
UNION
SELECT
  '2017-03-01' as first_day,
  '2017-03-31' as last_day),

cross_join as 
(SELECT * 
FROM subscriptions CROSS JOIN months
),

status AS
(
SELECT *,
 	CASE
  WHEN 
 		(subscription_start < first_day)
  	AND 
 		((subscription_end > first_day)
    OR (subscription_end IS NULL)) AND
  	segment = 87
 	THEN 1
	ELSE 0
  END as is_active_87,
  
CASE
  WHEN 
 		(subscription_start < first_day)
  	AND 
 		((subscription_end > first_day)
    OR (subscription_end IS NULL)) AND
  	segment = 30
 	THEN 1
	ELSE 0
  END as is_active_30
 ,
  
CASE
 WHEN 
 		(subscription_start < first_day)
 	AND 
		(subscription_end between first_day AND last_day) AND segment = 87
	THEN 1
	ELSE 0
 END as is_canceled_87
  ,
  
CASE
 WHEN 
 (subscription_start < first_day)
  AND 
	(subscription_end between first_day AND last_day) AND segment = 30
THEN 1
	ELSE 0
END as is_canceled_30

FROM cross_join)

SELECT first_day as month, sum(is_active_87),sum(is_active_30), sum(is_canceled_87), sum(is_canceled_30)
FROM status
group by month;

--obliczanie wspczynikw

WITH months AS
(SELECT
  '2017-01-01' as first_day,
  '2017-01-31' as last_day
UNION
SELECT
  '2017-02-01' as first_day,
  '2017-02-28' as last_day
UNION
SELECT
  '2017-03-01' as first_day,
  '2017-03-31' as last_day),

cross_join as 
(SELECT * 
FROM subscriptions CROSS JOIN months
),

status AS
(
SELECT *,
 	CASE
  WHEN 
 		(subscription_start < first_day)
  	AND 
 		((subscription_end > first_day)
    OR (subscription_end IS NULL)) AND
  	segment = 87
 	THEN 1
	ELSE 0
  END as is_active_87,
  
CASE
  WHEN 
 		(subscription_start < first_day)
  	AND 
 		((subscription_end > first_day)
    OR (subscription_end IS NULL)) AND
  	segment = 30
 	THEN 1
	ELSE 0
  END as is_active_30
 ,
  
CASE
 WHEN 
 		(subscription_start < first_day)
 	AND 
		(subscription_end between first_day AND last_day) AND segment = 87
	THEN 1
	ELSE 0
 END as is_canceled_87
  ,
  
CASE
 WHEN 
 (subscription_start < first_day)
  AND 
	(subscription_end between first_day AND last_day) AND segment = 30
THEN 1
	ELSE 0
END as is_canceled_30

FROM cross_join)

SELECT first_day as month, 1.0*sum(is_canceled_87)/sum(is_active_87), 1.0*sum(is_canceled_30)/sum(is_active_30)
FROM status
group by month;

--How would you modify this code to support a large number of segments?

WITH months AS
(SELECT
  '2017-01-01' as first_day,
  '2017-01-31' as last_day
UNION
SELECT
  '2017-02-01' as first_day,
  '2017-02-28' as last_day
UNION
SELECT
  '2017-03-01' as first_day,
  '2017-03-31' as last_day),
  

cross_join as 
(SELECT * 
FROM subscriptions CROSS JOIN months),

status AS
(SELECT *,
 	CASE
  WHEN 
 		(subscription_start < first_day)
  AND 
 		((subscription_end > first_day)
  OR (subscription_end IS NULL)) 
  THEN 1
	ELSE 0
  END as is_active,
  
CASE
  WHEN 
 		(subscription_start < first_day)
 	AND 
		(subscription_end between first_day AND last_day) 
	THEN 1
	ELSE 0
  END as is_canceled
  
FROM cross_join)

SELECT first_day as month, segment, 1.0*sum(is_canceled)/sum(is_active) AS churn_rate
FROM status
group by month, segment
;


-------------------------
Calculating Churn Rates
Four months into launching Codeflix, management asks you to look into subscription churn rates. It's early on in the business and people are excited to know how the company is doing.
The marketing department is particularly interested in how the churn compares between two segments of users. They provide you with a dataset containing subscription data for users who were acquired through two distinct channels.
The dataset provided to you contains one SQL table, subscriptions. Within the table, there are 4 columns:
id - the subscription id
subscription_start - the start date of the subscription
subscription_end - the end date of the subscription
segment - this identifies which segment the subscription owner belongs to
Codeflix requires a minimum subscription length of 31 days, so a user can never start and end their subscription in the same month.
If you get stuck during this project, check out the project walkthrough video which can be found at the bottom of the page after the final step of the project.

Tasks
10/10Complete
Mark the tasks as complete by checking them off
Get familiar with the data

1.
Take a look at the first 100 rows of data in the subscriptions table. How many different segments do you see?
Stuck? Get a hint


2.
Determine the range of months of data provided. Which months will you be able to calculate churn for?
Stuck? Get a hint

Calculate churn rate for each segment

3.
You'll be calculating the churn rate for both segments (87 and 30) over the first 3 months of 2017 (you can't calculate it for December, since there are no subscription_end values yet). To get started, create a temporary table of months.
Stuck? Get a hint


4.
Create a temporary table, cross_join, from subscriptions and your months. Be sure to SELECT every column. 
Stuck? Get a hint


5.
Create a temporary table, status, from the cross_join table you created. This table should contain:
id selected from cross_join
month as an alias of first_day
is_active_87 created using a CASE WHEN to find any users from segment 87 who existed prior to the beginning of the month. This is 1 if true and 0 otherwise.
is_active_30 created using a CASE WHEN to find any users from segment 30 who existed prior to the beginning of the month. This is 1 if true and 0 otherwise.
Stuck? Get a hint


6.
Add an is_canceled_87 and an is_canceled_30 column to the status temporary table. This should be 1 if the subscription is canceled during the month and 0 otherwise.
Stuck? Get a hint


7.
Create a status_aggregate temporary table that is a SUM of the active and canceled subscriptions for each segment, for each month.
The resulting columns should be:
sum_active_87
sum_active_30
sum_canceled_87
sum_canceled_30
Stuck? Get a hint


8.
Calculate the churn rates for the two segments over the three month period. Which segment has a lower churn rate?
Stuck? Get a hint

Bonus

9.
How would you modify this code to support a large number of segments?
