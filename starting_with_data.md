### Question 1: Find the top 3 products with the highest sentiment score? This will indicate a positive customer response.

**SQL Queries:**

--getting an idea of the types of columns in these tables
`select * from products `
`select * from analytics`
--need to convert sentimentScore to a float

```
SELECT NAME,
	SENTIMENTSCORE
FROM PRODUCTS
WHERE SENTIMENTSCORE IS NOT NULL
ORDER BY SENTIMENTSCORE DESC
LIMIT 3
```

**Answer:**

Users appear to have the most positive sentiment regarding "USB wired soundbar - in store only". Further
investigation may indicate satisfaction that can be replicated in other products.


### Question 2: What are the most popular main types of products are bought through 'Referral'?

**SQL Queries:**
```
WITH Cte_ReferralGroups AS (
    SELECT
        count(SPLIT_PART(v2productcategory, '/', 2)) AS count_product_category,
		SPLIT_PART(v2productcategory, '/', 2) AS mainCategory,
        v2productname,
        channelgrouping
    FROM all_sessions
    WHERE channelgrouping = 'Referral'
    GROUP BY v2productcategory,v2productname, channelgrouping
)

--select SPLIT_PART(v2productcategory, '/', 2) AS product_cat_clean from all_sessions

SELECT
    count_product_category AS ReferralCount,
	mainCategory,
    v2productname,
    channelgrouping
FROM Cte_ReferralGroups
GROUP BY count_product_category,mainCategory, v2productname, channelgrouping
ORDER BY ReferralCount DESC;
```

**Answer:**
It is a clear winner that Nest products are the most popular referred product as the top 7 items fall
into the main category of nest. From the query you can see the different nest products. Makes sense
to maximize this by having a referral product or bonus for this to keep customers happy.


### Question 3: During what periods (day,month,year) are people spending the most 'timeonsite'?

**SQL Queries:**
```
SELECT MAX(TIMEONSITE),
	TO_CHAR(date, 'Month') AS MONTH_NAME
FROM ANALYTICS
GROUP BY date
ORDER BY MAX(TIMEONSITE) DESC
```

**Answer:**

July seems to be the time when users spent the most time on site. Therefore it would be a good time to run promotions
outside of the usual sale periods i.e. Boxing day.


### Question 4: What is the country and product that has the most stock of an item?

**SQL Queries:**
```
SELECT A.COUNTRY,
	P.NAME as ProductName,
	MAX("stockLevel"::integer) as MostStock
FROM ALL_SESSIONS A
JOIN PRODUCTS P ON A.PRODUCTSKU = P."SKU"
GROUP BY A.COUNTRY,
	P.NAME
ORDER BY MAX("stockLevel"::integer) DESC
LIMIT 1;
```

**Answer:**
Water bottles have the most stock in the US with 19,675 units. And thus it could be wise to have a sale/promotion to get rid of some stock and free up inventory space.
