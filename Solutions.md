
**A. Customer Journey**

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

``` sql

SELECT s.customer_id,
       s.plan_id,
       p.plan_name,
       s.start_date
   FROM subscriptions s
      JOIN plans p ON p.plan_id  = s.plan_id 
   WHERE customer_id = 1 
     OR  customer_id = 2 
	   OR customer_id = 11 
	   OR customer_id = 13 
	   OR customer_id = 15 
	   OR customer_id = 16 
	   OR customer_id = 18 
	   OR customer_id = 19 
	 ORDER BY customer_id ASC

```

|customer_id |	plan_id  |	   plan_name    |	start_date  |
|------------|-----------|------------------|-------------|
|     1	     |      0	   |   trial	        | 2020-08-01  |
|     1	     |      1	   |   basic monthly	| 2020-08-08  |
|     2	     |      0	   |   trial	        | 2020-09-20  |
|     2	     |      3	   |   pro annual     | 2020-09-27  |
|     11	   |      0	   |   trial          | 2020-11-19  |
|     11	   |      4 	 |   churn          | 2020-11-26  |
|     13	   |      0	   |   trial          | 2020-12-15  |
|     13	   |      1	   |   basic monthly  |	2020-12-22  |
|     13	   |      2	   |   pro monthly    | 2021-03-29  |
|     15	   |      0 	 |   trial          | 2020-03-17  |
|     15	   |      2	   |   pro monthly    | 2020-03-24  |
|     15	   |      4	   |   churn          | 2020-04-29  |
|     16	   |      0	   |   trial          | 2020-05-31  |
|     16	   |      1	   |   basic monthly  | 2020-06-07  |
|     16	   |      3	   |   pro annual     | 2020-10-21  |
|     18	   |      0	   |   trial          | 2020-07-06  |
|     18	   |      2	   |   pro monthly    | 2020-07-13  |
|     19	   |      0	   |   trial          | 2020-06-22  |
|     19	   |      2	   |   pro monthly    | 2020-06-29  |
|     19	   |      3	   |   pro annual     | 2020-08-29  |

Customer 1 signed up for the trial and the following the trial downgraded to the basic monthly, instead of keeping the automatic pro monthly.

Customer 2 signed up for the trial and before the end of the trial upgraded to the pro annual.

Customer 11 signed up for the trial and cancelled after the trial expired.

Customer 13 signed up for the trial and the following the trial downgraded to the basic monthly, instead of keeping the automatic pro monthly. After a week on the basic monthly plan they upgraded up to pro monthly.

Customer 15  signed up for the trial and the following the trial stayed on the automatic pro monthly plan before cancelling 5 days later.

Customer 16 signed up for the trial and the following the trial downgraded to the basic monthly, instead of keeping the automatic pro monthly. After two weeks on basic monthly customer 16 upgraded to pro monthly.

Customer 18 signed up for the trial and the following the trial stayed on the automatic pro monthly plan.

Customer 19 signed up for the trial and the following the trial stayed on the automatic pro monthly plan. After two months on the pro monthly plan customer 19 upgraded to pro annual. 



**B. Data Analysis Questions**


How many customers has Foodie-Fi ever had?

```sql

SELECT COUNT(DISTINCT(customer_id)) as Total_customers
 FROM subscriptions
```
			 
| total_customers |
|-----------------|
|      1000       |



What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```SQL

SELECT trial_start_month,
       COUNT(Trial_start_month) as total_start_dates
        FROM (SELECT DATE_PART('Month', start_date) AS Trial_start_month
             FROM subscriptions
		    WHERE plan_id = 0)
			GROUP BY trial_start_month
			ORDER BY total_start_dates DESC
```

| trial_start_month |	 total_start_dates |
|-------------------|--------------------|
|         3	        |         94         |
|         7	        |         89         |
|         5	        |         88         |
|         8	        |         88         |
|         1	        |         88         |
|         9	        |         87         | 
|         12	      |         84         |
|         4	        |         81         |
|         6	        |         79         |
|         10	      |         79         |
|         11	      |         75         |
|         2	        |         68         |



What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```SQL

SELECT p.plan_name,
       COUNT(s.start_date) as post_2020_signups
   FROM subscriptions s
     JOIN plans p on s.plan_id = p.plan_id
       WHERE s.start_date >= '2021-01-01'
GROUP BY plan_name

```

|   plan_name   | post_2020_signups |
|---------------|-------------------|
|   pro annual  |	       63         |
|    churn	    |        71         |
|  pro monthly   |        60         |
| basic monthly |        8          |


What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```SQL
with total_cust as
      (SELECT COUNT(DISTINCT(customer_id)) AS total_cust
           FROM subscriptions),

total_churn AS (SELECT COUNT(DISTINCT(customer_id)) AS total_churn
                      FROM subscriptions
                      WHERE plan_id = 4 )

     SELECT total_churn,
            (100*total_churn::FLOAT/total_cust::FLOAT) AS percentage
                from total_churn, total_cust
```

| total_churn |	percentage |
|-------------|------------|
|     307	  |            30.7 |


How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```SQL 
with lead as (SELECT customer_id,
                     plan_id,
                     start_date, 
             LEAD(plan_id, 1) OVER (ORDER BY customer_id)
			  AS next_plan
        FROM subscriptions
   OEDER BY customer_id)
     
 
SELECT COUNT(customer_id) AS churned_customers,
            (COUNT(customer_id)*100)/(SELECT COUNT(DISTINCT(customer_id))
      FROM subscriptions) AS churn_percentage
FROM lead
     WHERE plan_id = 0 
        AND next_plan = 4
```

| churned_customers  |	  churn_percentage   |
|--------------------|-----------------------|
|   92	              |                 9 |


What is the number and percentage of customer plans after their initial free trial?

```SQL

with lead as (SELECT customer_id, plan_id, start_date, 
			  LEAD(plan_id, 1) over (order by customer_id)
			  AS next_plan
        FROM subscriptions
   ORDER BY customer_id)
     
 
SELECT plan_id, COUNT(customer_id) AS customer_total,
           ROUND((COUNT(customer_id)*100.0)/(SELECT COUNT(DISTINCT(customer_id))
      FROM subscriptions),1) AS customer_percentage
FROM lead
     WHERE plan_id != 0 
	 AND plan_id != 4 
	 AND next_plan != 4
		GROUP BY plan_id
		ORDER BY customer_percentage DESC
```

| plan_id   |	customer_total  |     customer_percentage    |
|-----------|-----------------|----------------------------|
|   1	     |            449	 |                                 44.9  |
| 2	       |          427	    |                              42.7   |
| 3	        |         252	     |                             25.2   |

 
What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```SQL
with plan_rank AS (SELECT customer_id,
                          start_date,
                          plan_id, 
                          RANK () OVER(PARTITION BY customer_id ORDER BY plan_id desc) AS plan_rank
                 FROM subscriptions 
                    WHERE start_date <= '2020-12-31')

SELECT plan_id,
       COUNT(customer_id), 
       ROUND(100.0 * COUNT(customer_id) /(SELECT COUNT(DISTINCT customer_id)
    FROM subscriptions),1) 
	     AS plan_percentage  
FROM plan_rank 
WHERE plan_rank = 1
GROUP BY plan_id

```

| plan_id  |	count   |	plan_percentage |
|----------|----------|-----------------|
| 0	       |       19	   |    1.9       |
| 1	       |       224	 |      22.4 |
| 2	       |       326	 |      32.6 |
| 3	       |       195	 |      19.5 |
| 4	       |       236	 |      23.6 |


How many customers have upgraded to an annual plan in 2020?

```SQL
with annual_subs AS (SELECT customer_id, 
		                        plan_id,
                            date_part('YEAR', start_date) as year_2020
                      FROM subscriptions
                      WHERE plan_id = 3)

SELECT COUNT(customer_id) AS annual_subs_for_2020
     FROM annual_subs 
     WHERE year_2020 = 2020
```

| annual_subs_for_2020 |
|----------------------|
|   195  |


How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

``SQL 
with annual_plans AS (SELECT customer_id, 
                             start_date, 
                             plan_id 
                    FROM subscriptions 
                    WHERE plan_id = 3),

trials AS (SELECT customer_id, 
                  start_date, 
                  plan_id
             FROM subscriptions
             WHERE plan_id = 0)
  
SELECT ROUND(AVG(p.start_date-t.start_date)) AS avg_signup_times_days
       FROM trials t 
       JOIN annual_plans p ON t.customer_id = p.customer_id

| avg_signup_times_days |
|-----------------------|
  |     105 |

Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

```SQL
with annual_plans AS (SELECT customer_id,
                             start_date,
                             plan_id 
                        FROM subscriptions 
                        WHERE plan_id = 3),

trials AS (SELECT customer_id,
                  start_date,
                  plan_id
            FROM subscriptions
            WHERE plan_id = 0),
  
days_upgrade AS (SELECT (WIDTH_BUCKET((p.start_date-t.start_date), 0,360,12)) AS days_upgrade 
FROM trials t 
JOIN annual_plans p ON t.customer_id = p.customer_id)

SELECT ((days_upgrade-1)*30) || '-' ||(days_upgrade*30) AS Days_to_upgrade,
COUNT(days_upgrade) AS Customers
FROM days_upgrade
GROUP BY days_upgrade
ORDER BY days_upgrade ASC

```

| days_to_upgrade |	customers |
|-----------------|-----------|
| 0-30            |               48  |
| 30-60           |     	   25 |
| 60-90	           |               33 |
| 90-120	          |     35  |
| 120-150	           |   43  |
| 150-180	            |  35 |
| 180-210	             | 27 |
| 210-240	        |      4 |
| 240-270	         |     5 |
| 270-300	          |    1 |
| 300-330	           |    1 |
| 330-360	            |  1 |

How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```SQL

with lead AS (SELECT customer_id,
                     plan_id,
                     start_date, 
			  LEAD(plan_id, 1) OVER (ORDER BY customer_id)
			  AS next_plan
        FROM subscriptions
   ORDER BY customer_id)
     
 SELECT COUNT(customer_id) AS downgrade_customers
    FROM lead
     WHERE plan_id = 2 
        AND next_plan = 1
	     AND start_date BETWEEN '2020-01-01' AND '2020-12-31';
```

| downgrade_customers |
|---------------------|
|     0   |


**C. Challenge Payment Question**

The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

monthly payments always occur on the same day of month as the original start_date of any monthly paid plan

upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately

upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period

once a customer churns they will no longer make payments

```sql

WITH RECURSIVE dates as (SELECT c.customer_id, 
                                c.plan_id, 
	                        p.plan_name, 
	                        p.price, 
	                        c.start_date,
	                        LEAD(c.start_date, 1) OVER(PARTITION BY c.customer_id) AS end_date
                             FROM subscriptions c
                          JOIN plans p ON c.plan_id = p.plan_id
                          WHERE start_date BETWEEN '2020-01-01' AND '2020-12-31'
                          AND c.plan_id != 0 
                          AND c.plan_id != 4
                           ORDER BY c.customer_id),
					  
        full_dates AS (SELECT customer_id, 
                              plan_id, 
	                      plan_name, 
	                      price, 
	                      start_date,
	                      COALESCE(end_date, '2020-12-31') AS end_date
                          FROM dates),
				
        full_dates2 AS (SELECT customer_id, 
                               plan_id, 
	                       plan_name, 
	                       price, 
	                       start_date,
                               end_date 
                           FROM full_dates
		
						UNION ALL		   

            SELECT customer_id, 
                   plan_id, 
                   plan_name, 
                   price, 
                   DATE(start_date + INTERVAL '1 MONTH') AS start_date,
                   end_date
               FROM full_dates2
               WHERE end_date > DATE(start_date + INTERVAL '1 MONTH') 
               AND plan_id != 3),

full_dates3 AS (SELECT *, 
                     LAG(plan_id, 1)OVER(PARTITION BY customer_id ORDER BY start_date) AS previous_plan,
                     LAG(price, 1)OVER(PARTITION BY customer_id ORDER BY start_date) AS previous_bill,
                     LAG(start_date, 1)OVER(PARTITION BY customer_ID ORDER BY start_date) AS previous_date
	          FROM full_dates2
	          ORDER BY customer_id, start_date)
			
SELECT customer_id, 
       plan_id, 
       plan_name, 
       (CASE WHEN previous_plan = 2 and plan_id = 3 THEN DATE(previous_date + INTERVAL '1 MONTH')
		ELSE start_date
		END) AS start_date,
       (CASE WHEN previous_plan = 1 AND (plan_id = 2 OR plan_id = 3) THEN price - previous_bill
		ELSE price
	   END) AS price
    FROM full_dates3
    ORDER BY customer_id, start_date;

```

This is a small sample of the output;

| customer_id |	plan_id | plan_name |	start_date |	price  |
|-------------|---------|-----------|--------------|-----------|
| 16 |	1 |	basic monthly |	2020-06-07 |	9.90 |
| 16 |	1 | 	basic monthly |	2020-07-07 |	9.90 |
| 16 |	1 |	basic monthly | 2020-08-07 |	9.90 |
| 16 |	1 |	basic monthly |	2020-09-07 |	9.90 |
| 16 |	1 |	basic monthly |	2020-10-07 |	9.90 |
| 16 |	3 |	pro annual |	2020-10-21 |	189.10 |
| 16 |	1 | 	basic monthly |	2020-11-07 |	9.90 |
| 16 |	1 |	basic monthly |	2020-12-07 |	9.90 |
| 17 |	1 | 	basic monthly |	2020-08-03 |	9.90 |
| 17 |	1 |     basic monthly |	2020-09-03 |	9.90 |
| 17 |	1 | 	basic monthly |	2020-10-03 |	9.90 |
| 17 |	1 |	basic monthly |	2020-11-03 |	9.90 |
| 17 |	1 |	basic monthly |	2020-12-03 |	9.90 |
| 17 |	3 |	pro annual |	2020-12-11 |	189.10 |
| 18 |	2 | 	pro monthly |	2020-07-13 |	19.90 |
| 18 |	2 |	pro monthly |	2020-08-13 |	19.90 | 
| 18 |	2 |	pro monthly |   2020-09-13 |	19.90 | 
| 18 |	2 |	pro monthly |	2020-10-13 |	19.90 |
| 18 |	2 |	pro monthly |	2020-11-13 |	19.90 |
| 18 |	2 |	pro monthly |	2020-12-13 |	19.90 |
| 19 |	2 | 	pro monthly |	2020-06-29 |	19.90 |
| 19 |	2 |	pro monthly |	2020-07-29 |	19.90 |
| 19 |	3 | 	pro annual |	2020-08-29 |	199.00 |

   

 




