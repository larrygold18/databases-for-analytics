# Exercise 05: SQLDA Database - Dates, Data Quality, Arrays, and JSON

- Name:
- Course: Database for Analytics
- Module:
- Database Used:  `sqlda` (Sample Datasets)
- Tools Used: PostgreSQL (pgAdmin or psql)

---

## Instructions

- Use the **sqlda** database from the "Loading the Sample Datasets" instructions.
- For each SQL task:
  - Include your SQL in a fenced code block
  - Execute it and include a **screenshot** showing the query and results
- Store screenshots in the `screenshots/` folder and embed them below each answer.
- For explanation questions:
  - Write your answer in complete sentences
  - Include a screenshot if requested

---

## Question 1

Using the `sqlda` database, write the SQL needed to show a **list of years** that emails were sent.

Your results should list years like this (order matters):

```
year
2011
2013
2014
2015
2016
2017
2018
2019
```

### SQL

```sql
-- SELECT DISTINCT
  EXTRACT(year FROM sent_date) AS year
FROM public.emails
ORDER BY year;

```

### Screenshot

![Q1 Screenshot](screenshots/q1_email_years.png)

---

## Question 2

Using the `sqlda` database, write the SQL needed to show the **number of messages sent by year**, ordered by year (as shown in the prompt).

Output should resemble:

```
count   year
...
```

### SQL

```sql
-- SELECT
  COUNT(*) AS count,
  EXTRACT(YEAR FROM sent_date) AS year
FROM emails
GROUP BY year
ORDER BY year;

```

### Screenshot

![Q2 Screenshot](screenshots/q2_message_count_by_year.png)

---

## Question 3

Using the `sqlda` database, write the SQL needed to show:
- the **sent date**
- the **opened date**
- the **interval** between the two

Only include emails that contain **both** a sent date and an opened date.

### SQL

```sql
-- SELECT
  sent_date,
  opened_date,
  opened_date - sent_date AS interval
FROM emails
WHERE sent_date IS NOT NULL
  AND opened_date IS NOT NULL
ORDER BY sent_date;

```

### Screenshot

![Q3 Screenshot](screenshots/q3_sent_opened_interval.png)

---

## Question 4

Using the `sqlda` database, write the SQL needed to show emails that contain an **opened date BEFORE the sent date**.

### SQL

```sql
-- SELECT
  email_id,
  sent_date,
  opened_date
FROM emails
WHERE opened_date IS NOT NULL
  AND sent_date IS NOT NULL
  AND opened_date < sent_date
ORDER BY sent_date;

```

### Screenshot

![Q4 Screenshot](screenshots/q4_opened_before_sent.png)

---

## Question 5

Using the `sqlda` database: there are **over 100 emails** that contain an opened date **BEFORE** the sent date.

After looking at the data, **why is this the case?**

### Answer

The emails appear to be opened before they were sent due to data quality issues, such as timestamps being logged from different systems or time zones and sent dates being populated or updated after open events. This is a common issue in sample and real-world datasets.

### Screenshot (if requested by instructor)

![Q5 Screenshot](screenshots/q5_explain_date_issue.png)

---

## Question 6

Using the `sqlda` database, explain in your own words what the following code does:

```sql
CREATE TEMP TABLE customer_points AS (
    SELECT
        customer_id,
        point(longitude, latitude) AS lng_lat_point
    FROM customers
    WHERE longitude IS NOT NULL
    AND latitude IS NOT NULL
);

CREATE TEMP TABLE dealership_points AS (
    SELECT
        dealership_id,
        point(longitude, latitude) AS lng_lat_point
    FROM dealerships
);

CREATE TEMP TABLE customer_dealership_distance AS (
    SELECT
       customer_id,
       dealership_id,
       c.lng_lat_point <@> d.lng_lat_point AS distance
    FROM customer_points c
    CROSS JOIN dealership_points d
);
```

### Answer

The script creates temporary tables of geographic points for customers and dealerships, then computes the pairwise distance between every customer and every dealership:

customer_points: keeps customer_id and a geometric point(longitude, latitude) for customers that have coordinates.

dealership_points: same for dealerships.

customer_dealership_distance: does a Cartesian product (every customer × every dealership) and computes c.lng_lat_point <@> d.lng_lat_point (a distance operator) to produce a table of distances between each customer and each dealership.

---

## Question 7

Using the `sqlda` database, write SQL to display an **array of salespeople for each dealership**, sorted by dealership.

For example - dealership 1 is below:

```text
"{""Fidell,Granville"",""Onele,Jereme"",""Sheriff,Lelia"",""McSpirron,Massimiliano"",""Rennick,Nadia"",""Mace,Eveleen"",""Oxteby,Dukie"",""Spong,Marcos"",""Wogden,Quent"",""Duny,Sandye"",""Loraine,Englebert"",""Meere,Ira"",""Gibbens,Cristine"",""Prine,Lyda"",""McCoughan,Sheff"",""Schule,Giselbert"",""McAndie,Eleen"",""Dosedale,Dorie"",""Nafziger,Shay""}"
```

### SQL

```sql
-- SELECT
    dealership_id,
    ARRAY_AGG(last_name || ',' || first_name ORDER BY last_name, first_name) AS salespeople
FROM salespeople
GROUP BY dealership_id
ORDER BY dealership_id;
```

### Screenshot

![Q7 Screenshot](screenshots/q7_salespeople_array_by_dealership.png)

---

## Question 8

Using the `sqlda` database, write SQL to display:
- an **array of salespeople for each dealership**
- the **state** of the dealership
- the **number of salespeople** for the dealership

Sort by **state**.

Reference image:

![05-ExerciseArray](./instructions/05-ExerciseArray.jpg)

### SQL

```sql
SELECT d.dealership_id,
d.state,
COUNT(*) AS count,
ARRAY_AGG(
(s.last_name || ',' || s.first_name)
ORDER BY s.salesperson_id
) AS array_agg  
FROM dealerships d
JOIN salespeople s
ON d.dealership_id = s.dealership_id
GROUP BY d.dealership_id, d.state
ORDER BY d.state,count;
```

### Screenshot

![Q8 Screenshot](screenshots/q8_salespeople_array_state_count.png)

---

## Question 9

Using the `sqlda` database, write the SQL needed to convert the **customers** table to **JSON**.

### SQL

```sql
-- SELECT row_to_json(c)
   AS customer_json
   FROM customers c;

```

### Screenshot

![Q9 Screenshot](screenshots/q9_customers_to_json.png)

---

## Question 10

Using the `sqlda` database, write SQL to display:
- an **array of salespeople for each dealership**
- the **state**
- the **number of salespeople**
- sorted by **state**

Then **convert this result to JSON**.

Reference image:

![05-ExerciseArray-1](./instructions/05-ExerciseArray-1.jpg)

### SQL

```sql
-- SELECT
  json_build_object(
    'dealership_id', d.dealership_id,
    'state', d.state,
    'num_salespeople', COUNT(s.*),
    'salespeople', json_agg((s.last_name || ', ' || s.first_name) ORDER BY s.salesperson_id)
  ) AS dealership_json
FROM dealerships d
JOIN salespeople s
  ON d.dealership_id = s.dealership_id
GROUP BY d.dealership_id, d.state
ORDER BY d.state, COUNT(s.*);
```

### Screenshot

![Q10 Screenshot](screenshots/q10_salespeople_array_to_json.png)
