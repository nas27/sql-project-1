### Part 3: Starting with Questions

**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

**SQL Queries:**

```
WITH HIGH_REV_SESSIONS AS
	(SELECT CITY,
			COUNTRY,
			SUM(ROUND(TOTALTRANSACTIONREVENUENEW,

								2)) AS TOTAL_SUM
		FROM ALL_SESSIONS
		WHERE CITY IS NOT NULL
			AND COUNTRY IS NOT NULL
			AND TOTALTRANSACTIONREVENUENEW IS NOT NULL
		GROUP BY CITY,
			COUNTRY)
SELECT CITY,
	COUNTRY,
	TOTAL_SUM
FROM HIGH_REV_SESSIONS
ORDER BY TOTAL_SUM DESC
LIMIT 5
```

>**Answer:** Cities in the United States have the highest level of transaction revenues. These are mostly located in California
with the exception of Atlanta. The fifth location not in the US is Tel Aviv Israel.

**Question 2: What is the average number of products ordered from visitors in each city and country?**

**SQL Queries:**
```
SELECT PRODUCTSKU,
	ALS.CITY,
	ALS.COUNTRY,
	AVG(P.ORDEREDQUANTITY)
FROM ALL_SESSIONS ALS
JOIN PRODUCTS P ON ALS.PRODUCTSKU = P."SKU"
GROUP BY P.ORDEREDQUANTITY,
	PRODUCTSKU,
	ALS.CITY,
	ALS.COUNTRY
	ORDER BY AVG(P.ORDEREDQUANTITY) DESC;
```

>**Answer:** Since there are a number of different cities it returned many different averaged. On average, U.S ordered the most amount of products.


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

**SQL Queries:**
--below query allows me to explore the data and see where i can best attain this required data 
```
SELECT TABLE_NAME,
	COLUMN_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME like '%category%';
```

--below query counts the categories and orders desc so we can see the categories that occurred the most.
```
SELECT COUNT(V2PRODUCTCATEGORY),
	V2PRODUCTCATEGORY,
	COUNTRY,
	CITY
FROM ALL_SESSIONS
WHERE V2PRODUCTCATEGORY IS NOT NULL
GROUP BY COUNTRY,
	CITY,
	V2PRODUCTCATEGORY
HAVING COUNT(V2PRODUCTCATEGORY) > 1
ORDER BY COUNT(V2PRODUCTCATEGORY) DESC
```

>**Answer:** Visitors in the U.S. mostly order Home products.

**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

**SQL Queries:** 

--I started with this basic query that I wrote myself:

```
SELECT "productSKU",
	ALS.V2PRODUCTNAME,
	TOTAL_ORDERED
FROM SALES_REPORT SR
JOIN ALL_SESSIONS ALS ON ALS."productsku" = SR."productSKU"
```

> I wanted to explore AI and how it can be used to streamline and aid the process. So I then had ChatGPT transform it into a ranked CTE and made sure I understood the logic by asking it questions to confirm my understanding:


Cite: _OpenAI. (2023). ChatGPT (10 28 version) [Large language model]. https://chat.openai.com_
```
WITH RankedProducts AS (
    SELECT
        als.city,
        als.country,
        sr."productSKU",
        als.v2productname,
        total_ordered,
        RANK() OVER (PARTITION BY als.city, als.country ORDER BY total_ordered DESC) AS ranking
    FROM sales_report sr
    JOIN all_sessions als ON als."productsku" = sr	."productSKU"
)
SELECT
    city,
    country,
    "productSKU",
    v2productname,
    total_ordered
FROM RankedProducts
WHERE ranking = 1;
```
```
SELECT distinct("productSKU"),
	ALS.V2PRODUCTNAME,
	TOTAL_ORDERED, city, country
FROM SALES_REPORT SR
JOIN ALL_SESSIONS ALS ON ALS."productsku" = SR."productSKU"
group by distinct("productSKU"),ALS.V2PRODUCTNAME,TOTAL_ORDERED,city, country
order by TOTAL_ORDERED desc;
```


>**Answer:** Ballpoint pens in many cities in the U.S.A including Boston, Mountain View, Palo Alto, San Francisco, Santa Clara, and Seattle.
Next we have a variety of sport bottles in 

**Question 5: Can we summarize the impact of revenue generated from each city/country?**

**SQL Queries:**

```
SELECT  COUNTRY,
	SUM(ROUND("totaltransactionrevenuenew",2)) AS TOTAL_REVENUE
FROM ALL_SESSIONS
WHERE TOTALTRANSACTIONREVENUENEW IS NOT NULL
	AND COUNTRY IS NOT NULL
GROUP BY COUNTRY
ORDER BY TOTAL_REVENUE DESC;

SELECT CITY,
	SUM(ROUND("totaltransactionrevenuenew",2)) AS TOTAL_REVENUE
FROM ALL_SESSIONS
WHERE TOTALTRANSACTIONREVENUENEW IS NOT NULL
AND COUNTRY = 'United States'
	AND CITY IS NOT NULL
GROUP BY CITY
ORDER BY TOTAL_REVENUE DESC;
```

>**Answer:** All of the cities are from the United States. It seems these cities have the most impact on total revenue. The top city is "San Francisco".
