What issues will you address by cleaning the data?

* PK Constraints
* Duplicates
* Null values
* Invalid values
* Inefficient values i.e. (not set) text

### Queries:
## Data Cleaning

```
UPDATE your_table
SET unit_price = unit_price / 1000000;
```

This resulted in an error because of the fact that I had to create everything as a varchar because the data
was not very clean.

**issue LINE 2: SET "unit_price" = "unit_price" / 1000000;
ERROR:  operator does not exist: character varying / integer**

#### Steps to fix: 

 Created new col and tried to cast as integer
took a while to process but was successful.

--create new column

```ALTER TABLE analytics ADD "unit_price2" INTEGER;```

--transfer data to new col 

``` UPDATE analytics
SET "unit_price2" = CAST(unit_price AS INTEGER); 
```

--verify similar data exists

``` 
select * from analytics limit 10
```

--update new col with fix unit_price

```
UPDATE analytics
SET "unit_price2" = "unit_price2" / 1000000;
```

-- I realized integer isnt the best type as it truncated the data, checked northwind and used their datatype float(10)
--repeated above steps with new float col


-- **Success** so i will create new col and drop the old one
--drop unnecessary columns
```
ALTER TABLE your_table DROP COLUMN your_column;
```

--got error consulted chatggpt and gave me query to clear cache
`SELECT pg_stat_statements_reset();`

--rename original col and rename my new col to what i want, drop original col
```
ALTER TABLE analytics
RENAME COLUMN unit_price TO unit_price_og;
```

`ALTER TABLE analytics RENAME COLUMN unit_price3 TO unit_price;`

--ran into issues importing csv, columnns blank
--drop table and recreate with sql

--attempting to clean the Time field and make sense of it
```
SELECT count(*) as count, CHAR_LENGTH(time) AS varchar_length
FROM all_sessions group by CHAR_LENGTH(time) order by varchar_length;
```

> In the above i am trying to uncover a pattern to see if a certain timestamp or format jumps out to me as frequent

--attempting to get most frequent transaction revenues i realize there are a lot of nulls that need to be cleaned before i can add a primary key
i was unable to add pkey as values were not unique

-explore duplicates
`SELECT fullvisitorid FROM all_sessions`
--this returns 15134 records
`SELECT DISTINCT(fullvisitorid) FROM all_sessions`
--this returns 14223 which means there is a discrepency with 911 erronous records


--going to make visitid PK because it has less duplicates

> I wanted to test out chatgpt functionality, I used chatgpt to help create a CTE. The following deleted records 1130 from the table
the idea to delete duplicates is based on the video provided by LHL: https://www.youtube.com/watch?v=iF6zHuReEZQ&t=223s
```
WITH ranked_sessions AS (
-- assigning a row number to each row based on the visitid and ordering by date in descending order
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY visitid ORDER BY date DESC) AS rn
    FROM
        all_sessions
)

--delete duplicate rows from the all_sessions table, leaving only the row with the latest date for each unique visitid.
DELETE FROM all_sessions
WHERE (visitid, date) IN (
    SELECT visitid, date
    FROM ranked_sessions
    WHERE rn > 1
);

--After this i was successfully able to add visitid as the PK

```

`ALTER TABLE all_sessions ADD PRIMARY KEY ("visitid"); `


--cities and countries "not set"
First i want to convert these to null instead to take up less space and make it easier to filter them out in my query

i used the below queries to data explore:

`SELECT * FROM all_sessions where city like '%not%'`

`SELECT * FROM all_sessions 
where country like '%not%'`

```
UPDATE all_sessions
SET country = NULL
WHERE country = '(not set)';
```
--same done for city field

```
UPDATE all_sessions
SET city = NULL
WHERE city = 'not available in demo dataset';
```

--question 2 orderedquantity, cannot perform agg function on it, cleaning needed to convert to appropropriate data type


`UPDATE PRODUCTS
SET "orderedQuantity" = CAST("orderedQuantity" AS integer);`


`ALTER TABLE PRODUCTS ADD ORDEREDQUANTITY2 integer;`


```
UPDATE PRODUCTS
SET ORDEREDQUANTITY2 = "orderedQuantity"::integer;
```


`ALTER TABLE PRODUCTS RENAME ORDEREDQUANTITY2 TO ORDEREDQUANTITY;`

--clean up categories for question 3
```
UPDATE all_sessions
SET v2productcategory = NULL
WHERE v2productcategory = '(not set)';
```

Below I performed **data exploration** to see if these fields could provide something useful to find the "top selling" products

`select v2productname,itemquantity,itemrevenue,"productrevenue" from all_sessions where productrevenue is not null`

>Finding: all null values therefore not usable
--from below query i have decided to utilize the below fields to help find the 'top selling' products.

`select "productSKU", total_ordered from sales_report;`

--total ordered needs to be an int
`UPDATE sales_report SET total_ordered = total_ordered::integer;`

--this returned successfully but didnt seem to work, i will attempt to create a new column as int, and populate it using CAST function then delete and rename it
```
ALTER TABLE sales_report
ADD COLUMN total_ordered2 INT;
UPDATE sales_report
SET total_ordered2 = CAST(total_ordered AS INT);
ALTER TABLE sales_report
DROP COLUMN total_ordered;
ALTER TABLE sales_report
RENAME COLUMN total_ordered2 TO total_ordered;
```


--convert sentimentScore so that i can't run aggregate function in my questions

`ALTER TABLE products
ADD COLUMN sentimentScoreFloat FLOAT;`

`UPDATE products
SET sentimentScoreFloat = CAST("sentimentScore" AS FLOAT);`

--verify update was successful with a select statement

`select sentimentScoreFloat from products`

`ALTER TABLE products
DROP COLUMN "sentimentScore";`

`ALTER TABLE products
RENAME COLUMN sentimentScoreFloat TO sentimentScore;`

--clean up the date column from all_sessions
`ALTER TABLE all_sessions
ADD new_date_column date;`

`UPDATE all_sessions
SET new_date_column = TO_DATE("date", 'YYYYMMDD');`

`select new_date_column from all_sessions`

```
ALTER TABLE all_sessions
DROP COLUMN "date";
```

--clean up timeonsite col
```ALTER TABLE analytics
ADD timeonsite_new INT;
```


`UPDATE analytics
SET timeonsite_new = CAST("timeonsite" AS INT);`

`select timeonsite_new from analytics`

`ALTER TABLE analytics
DROP COLUMN "timeonsite";`


`ALTER TABLE analytics
RENAME COLUMN timeonsite_new TO timeonsite;`


--get a picture of the duped ids
select fullvisitorid, count(fullvisitorid) as NonUniquePK
from all_sessions
group by fullvisitorid
having count(fullvisitorid) > 1;

--delete the extra fullvisitorids

DELETE FROM all_sessions
WHERE fullvisitorid IN (
	SELECT fullvisitorid FROM all_sessions
	GROUP BY fullvisitorid
	HAVING COUNT(fullvisitorid) > 1)
	
--you need to have the analytics data attached to a user
DELETE
FROM ANALYTICS
WHERE USERID IS NULL

--Attempting to fix some of the relationships
```
ALTER TABLE PRODUCTS ADD CONSTRAINT PK_SKU PRIMARY KEY ("SKU")
ALTER TABLE SALES_BY_SKU ADD CONSTRAINT PK_S_SKU PRIMARY KEY ("productsku")
ALTER TABLE SALES_REPORT ADD CONSTRAINT PK_SR_SKU PRIMARY KEY("productSKU")
ALTER TABLE ANALYTICS ADD CONSTRAINT FK_ANALYTICS_SESSION
FOREIGN KEY ("visitNumber")
```

--attempting to add a relationship but it failed
```
ALTER TABLE all_sessions
ADD CONSTRAINT fk_v_id
FOREIGN KEY ("fullvisitorid")
REFERENCES analytics(fullVisitorId);
```
