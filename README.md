# The-SQL-Murder-Mystery
The SQL Murder Mystery is an intercative SQL game that allows you to learn and practice SQL simultaneously. To access the game click **[here](https://mystery.knightlab.com/)**

Prior to querying lets acknowldge some key details that will help us find the murderer. 

### **Details**
- Crime: **Murder**
- Location: **SQL City**
- Date: **January 15, 2018**

### **Schema Diagram**

![Table](https://mystery.knightlab.com/schema.png)

The schema diagram displays the primary keys and column of each table that we'll be using during our investigation.

### Searching for the murder on January 15, 2018 

Run a query using the details above to filter data in the 'crime_scene_report' table

~~~ SQL 
SELECT *
FROM 
	crime_scene_report
WHERE
	type = "murder"
	
	AND
	
	city = "SQL City"
	
	AND 
	
	date  = 20180115
~~~
Result:

|date|type|description|city|
|---|---|---|---|
|20180115|murder|Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".|SQL City

### Finding the Witnesses 

~~~ SQL 

SELECT *
FROM 
	person
WHERE
	address_street_name = "Northwestern Dr"
ORDER BY 
	address_number DESC --the highest number is the last address on Northwestern Dr.
LIMIT 1 
~~~
Result:
|id|name|license_id|address_number|address_street_name|ssn|
|---|---|---|---|---|---|
|14887|Morty Schapiro|118009|4919|Northwestern Dr|111564949|

~~~ SQL 
SELECT *
FROM 
	person
WHERE 
	 name LIKE 'Annabel %' -- name beginning with 'Annabel'
~~~
Result:
|id|name|license_id|address_number|address_street_name|ssn|
|---|---|---|---|---|---|
|16371|Annabel Miller|490173|103|Franklin Ave|318771143|

### Interviews
Access the two witnesses' recounts of the crime . To this we have to JOIN the ‘Person’ and ‘Interview’ tables using the primary keys ‘person_id’ in the ‘Interview’ table and the ‘id’ in the ‘Person’ Table.

~~~ SQL
SELECT 
	p.name, i.transcript
FROM 
	person AS p --creating an alias for the ‘Person’ table to prevent syntax errors
JOIN Interview AS i ON p.id = i.person_id -- join both tables using the primary keys in each table
WHERE 
	p.name = "Annabel Miller" OR p.name = "Morty Schapiro"
~~~
Result:
|name|transcript|
|---|---|
|Morty Schapiro|I heard a gunshot and then saw a man run out. He had a 'Get Fit Now Gym' bag. The membership number on the bag started with '48Z'. Only gold members have those bags. The man got into a car with a plate that included "H42W".|
|Annabel Miller|I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.|

### Verifying Witness Interviews 

To verify the witnessess interviews we’re going to make a nested query to work with a smaller dataset:

Use the following query to return the data that meet the conditions in which the membership status is gold and has a membership starting with 48Z

~~~ SQL 
SELECT *
FROM
	get_fit_now_member
WHERE 
	membership_status = "gold" AND  id LIKE '48Z%'
~~~
Result:
|id|person_id|name|membership_start_date|membership_status|
|---|---|---|---|---|
|48Z7A|28819|Joe Germuska|20160305|gold|
|48Z55|67318|Jeremy Bowers|20160101|gold|

This narrows down the dataset considerably. Now, let's apply other conditions to this table to find the murderer:

To do this were going to make the previous query a subquery by putting parentheses around the first query and assigning it an alias.

~~~ SQL
(SELECT *
FROM
	get_fit_now_member
WHERE 
	membership_status = "gold" AND  id LIKE '48Z%') AS t -- alias
~~~
This subquery will be used in the FROM portion of the outer query

Next begin the outer query, and choose the column names indicated in the results of Morty Schapiro’s interview. To do this we need to JOIN 'get_fit_now_memeber' table with the 'person' and 'drivers_license' tables using primary keys indicated in each table:

~~~ SQL 
SELECT *

FROM
 (SELECT *
FROM
	get_fit_now_member
WHERE 
	membership_status = "gold" AND  id LIKE '48Z%') AS t
JOIN person AS p ON t.person_id = p.id
JOIN drivers_license AS d ON p.license_id = d.id
~~~
To narrow the scope of the results, enter specific column names in the SELECT statement of the outer query. Follow the steps below:

~~~ SQL
SELECT 
	t.name, t.id, t.membership_status, d.plate_number 
FROM 
(SELECT *
FROM
	get_fit_now_member
WHERE 
	membership_status = "gold" AND  id LIKE '48Z%') AS t
JOIN person AS p ON t.person_id = p.id
JOIN drivers_license AS d ON p.license_id = d.id
~~~
Results:
|name|id|memberhsip_status|plate_number|
|---|---|---|---|
|Jeremy Bowers|48Z55|gold|0H42W2|

To validate my answer I entered the name of the murderer in the INSERT statement

Result:
|value|
|---|
|Congrats, you found the murderer! But wait, there's more... ***If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime***. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.|

### Querying the interview transcript of Jeremy Bowers
Find Jeremy Bowers Interview transcript. To do so, we are going to JOIN the interview and person table using the primary key ‘person_id’ in the interview table and ‘id’ in the person table as well as adding a condition in the WHERE clause
~~~ SQL
SELECT 
	p.name, i.transcript
FROM
	interview AS i
JOIN person AS p ON i.person_id = p.id
WHERE 
  p.name = "Jeremy Bowers"
~~~
Result:
|name|transcript|
|---|---|
|Jeremy Bowers|I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.|

### Verifying Jeremy's interview

To verify the interview we are going JOIN several tables, drivers_license, person, income, facebook_event_checkin, using primary keys. In the WHERE statment we are going to filter by specific details given in the interview. 
~~~ SQL 
SELECT 
	p.name, d.gender, d.car_make, d.car_model,i.annual_income, COUNT(person_id) AS event_attended_ct
FROM drivers_license AS d
JOIN person AS p ON p.license_id = d.id
JOIN income AS i ON i.ssn = p.ssn
JOIN facebook_event_checkin AS f ON f.person_id = p.id
WHERE 
	height BETWEEN 65 AND 67
	
	AND 
	
	gender = "female"
	
	AND 
	
	car_make = "Tesla" 
ORDER BY i.annual_income DESC 
~~~

Result:
|name|gender|car_make|car_model|annual_income|event_attended_ct|
|---|---|---|---|---|---|
|Miranda Priestly|female|Tesla|Model S|310000|3|

To validate my answer I entered the name of the murderer in the INSERT statement

|value|
|---|
|Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!|
