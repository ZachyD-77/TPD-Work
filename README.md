# TPD-Work
Simple Join:
SELECT *
FROM users
LEFT JOIN premium_users
	ON users.id = premium_users.user_id
WHERE premium_users.user_id IS NULL;

Good Case Statement:
SELECT premium_users.user_id,
  months.months,
  CASE
    WHEN (
      premium_users.purchase_date <=
             months.months
      )
      AND
      (
        (premium_users.cancel_date >=
                months.months)
        OR
        premium_users.cancel_date IS NULL
      )
    THEN 'active'
    ELSE 'not_active'
  END as status

FROM premium_users
CROSS JOIN months;

Select INCI_ID, DATE_OCCU, Offense, statutdesc, agency
From vectorsde.gisdata.POLICE_INCIDENTS
--where (Offense in ('2101', '3301', '3311')) 
--or (Offense like '30%')
where Offense='3011'
and (DATE_OCCU like '2022%')
and (agency='TPD')
order by DATE_OCCU asc

SELECT users.username, COUNT(posts.id) as 'Posts_Made'
FROM users
LEFT JOIN
posts ON users.id = posts.user_id
GROUP BY users.id
ORDER BY 2 DESC

Select project_name as 'Project Name', Count(current_project) as 'Count of Project'
From projects
Inner Join employees
On projects.project_id = employees.current_project
Where current_project IS NOT NULL
Group By current_project
Having Count(current_project) > 1;

Select (count(*) * 2) - (
  Select Count(*)
  From employees
  Where current_project Is Not Null 
    And position = 'Developer') as 'Count'
From projects;

Sets a count of crime type, by crime type:
Select crime_type as 'Crime Type', count(crime_type) as 'Count of Crime Type'

From case_reports

Group By crime_type

Order By count(crime_type) desc

Shows Imcompatible Personality Types:
SELECT last_name, first_name, personality, project_name,
CASE 
   WHEN personality = 'INFP' 
   THEN (SELECT COUNT(*)
      FROM employees 
      WHERE personality IN ('ISFP', 'ESFP', 'ISTP', 'ESTP', 'ISFJ', 'ESFJ', 'ISTJ', 'ESTJ'))
   WHEN personality = 'ISFP' 
   THEN (SELECT COUNT(*)
      FROM employees 
      WHERE personality IN ('INFP', 'ENTP', 'INFJ'))
   -- ... etc.
   ELSE 0
END AS 'IMCOMPATS'
FROM employees
LEFT JOIN projects on employees.current_project = projects.project_id;

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
  '2017-03-31' as last_day
),
cross_join AS
(SELECT *
FROM subscriptions
CROSS JOIN months),
status As 
(Select id, first_day as month,
Case 
  When (subscription_start < first_day)
    And (
      subscription_end > first_day
      Or subscription_end Is Null
    ) Then 1 
  Else 0
End as is_active
From cross_join)
SELECT *
FROM status
LIMIT 100;

Final Churn Rate Query:
WITH months AS (
  SELECT 
    '2017-01-01' AS first_day, 
    '2017-01-31' AS last_day 
  UNION 
  SELECT 
    '2017-02-01' AS first_day, 
    '2017-02-28' AS last_day 
  UNION 
  SELECT 
    '2017-03-01' AS first_day, 
    '2017-03-31' AS last_day
), 
cross_join AS (
  SELECT *
  FROM subscriptions
  CROSS JOIN months
), 
status AS (
  SELECT 
    id, 
    first_day AS month, 
    CASE
      WHEN (subscription_start < first_day) 
        AND (
          subscription_end > first_day 
          OR subscription_end IS NULL
        ) THEN 1
      ELSE 0
    END AS is_active, 
    CASE
      WHEN subscription_end BETWEEN first_day AND last_day THEN 1
      ELSE 0
    END AS is_canceled 
  FROM cross_join
), 
status_aggregate AS (
  SELECT 
    month, 
    SUM(is_active) AS active, 
    SUM(is_canceled) AS canceled 
  FROM status 
  GROUP BY month
) 
SELECT
  month, 
  1.0 * canceled / active AS churn_rate 
FROM status_aggregate;

WINDOWS FUNCTION:

SELECT 
   month,
   change_in_followers,
   SUM(change_in_followers) OVER (
      ORDER BY month
   ) AS 'running_total',
   AVG(change_in_followers) OVER (
      ORDER BY month
   ) AS 'running_avg',
   COUNT(change_in_followers) OVER (
      ORDER BY month
   ) AS 'running_count'
FROM
   social_media
WHERE
   username = 'instagram';

SELECT username,
month,
change_in_followers,
    SUM(change_in_followers) OVER (
      Partition By username
      Order By month
    ) 'running_total_followers_change'
FROM social_media;

SELECT username,
   posts,
   LAST_VALUE (posts) OVER (
      PARTITION BY username 
      ORDER BY posts
      RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
   ) most_posts
FROM social_media;

WINDOWS FUNCTION RANK:

SELECT 
   RANK() OVER ( partition by week
      ORDER BY streams_millions desc
   ) AS 'rank', 
   artist, 
   week,
   streams_millions
FROM
   streams;

SELECT date, (CAST(high as 'REAL') +
CAST(low AS 'REAL')) / 2.0 AS 'average'
FROM weather;

SELECT purchase_id, DATE(purchase_date, '+7 days') as 'Last Refund Day'
FROM purchases;

SELECT  STRFTIME('%H', purchase_date)
FROM purchases;

SELECT STRFTIME('%m-%d', purchase_date) as 'reformatted'
FROM purchases
