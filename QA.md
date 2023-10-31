### Part 5: QA

The goal to maintain a quality of accuracy and reliability.

From my initial data exploration phase I was skeptical about the data and consistency of it as I saw lots of
data that simply didn't look right as compared to normalized and workable databases we have used thus far like Northwind and Chinook.

I noticed a ton of sanity issues. In the real world, it might be useful to find out where the source of data is an employ
some of input sanitation checks before allowing it to be inserted into wherever the csv file came from.

**Issues:**
+ rows containing nulls,
+ redundant fields 
+ duplicate records in supposed primary key probable invalid data just from the naked eye.

### Data assertion (Queries to hunt for problems)
**PASS - return 0**

**FAIL -  return >= 1**

```
select fullvisitorid ,
	visitid
	FROM all_sessions
	where fullvisitorid is null;
```
--returned 0

```
select fullvisitorid ,
	visitid
	FROM all_sessions
	where visitid is null;
```
	
--returned 0

--PASS
	
```
select "SKU",
	name
	FROM products
	where "SKU" is null;
```
--returned 0

```
select "SKU",
	name
	FROM products
	where "name" is null;
```
--returned 0

--PASS

```
select "productsku"	
	FROM sales_by_sku
	where "productsku" is null;
```
--returned 0

```
SELECT "productSKU"
FROM SALES_REPORT
WHERE "productSKU" IS NULL;
```

--returned 0

--PASS

#### Check email address/contact info

There doesn't seem to be any contact or user details yet there is a userid.

`SELECT COUNT(userid) FROM analytics WHERE userid IS NULL`

#### Rerun to QA ensure count is 1 after deletion of duplicated keys
```
select fullvisitorid, count(fullvisitorid) as NonUniquePK
from all_sessions
group by fullvisitorid
```

--PASS returned 1 count
