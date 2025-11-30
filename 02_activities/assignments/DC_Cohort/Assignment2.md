# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:
* Submission Due Date: `November 12, 2025`
* Weight: 70% of total grade
* The branch name for your repo should be: `assignment-two`
* What to submit for this assignment:
    * This markdown (Assignment2.md) with written responses in Section 1 and 4
    * Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
    * One .sql file 
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-two`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

***

## Section 1:
You can start this section following *session 1*, but you may want to wait until you feel comfortable wtih basic SQL query writing. 

Steps to complete this part of the assignment:
- Design a logical data model
- Duplicate the logical data model and add another table to it following the instructions
- Write, within this markdown file, an answer to Prompt 3


###  Design a Logical Model

#### Prompt 1
Design a logical model for a small bookstore. 📚

At the minimum it should have employee, order, sales, customer, and book entities (tables). Determine sensible column and table design based on what you know about these concepts. Keep it simple, but work out sensible relationships to keep tables reasonably sized. 

Additionally, include a date table. 

There are several tools online you can use, I'd recommend [Draw.io](https://www.drawio.com/) or [LucidChart](https://www.lucidchart.com/pages/).

**HINT:** You do not need to create any data for this prompt. This is a conceptual model only. 

<img src="./sql_assignment_2_prompt_1.png" width="600">

#### Prompt 2
We want to create employee shifts, splitting up the day into morning and evening. Add this to the ERD.
<img src="./sql_assignment_2_prompt_2.png" width="600">

#### Prompt 3
The store wants to keep customer addresses. Propose two architectures for the CUSTOMER_ADDRESS table, one that will retain changes, and another that will overwrite. Which is type 1, which is type 2? 

**HINT:** search type 1 vs type 2 slowly changing dimensions. 


<img src="./sql_assignment_2_prompt_3.png" width="300">

The type 1 table contains only one entry per customer address which is overwritten each time it is updated. The type 2 table contains start and end dates and an is_current column so that a new entry is added each time a customer address is changed instead of overwriting it.

***

## Section 2:
You can start this section following *session 4*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question


### Write SQL

#### COALESCE
1. Our favourite manager wants a detailed long list of products, but is afraid of tables! We tell them, no problem! We can produce a list with all of the appropriate details. 

Using the following syntax you create our super cool and not at all needy manager a list:
```
SELECT 
product_name || ', ' || product_size|| ' (' || product_qty_type || ')'
FROM product
```

But wait! The product table has some bad data (a few NULL values). 
Find the NULLs and then using COALESCE, replace the NULL with a blank for the first column with nulls, and 'unit' for the second column with nulls. 

**HINT**: keep the syntax the same, but edited the correct components with the string. The `||` values concatenate the columns into strings. Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. All the other rows will remain the same.

```
SELECT product_name || ', ' || COALESCE(product_size, '')|| ' (' || COALESCE(product_qty_type, 'unit') || ')'
FROM product
```


<div align="center">-</div>

#### Windowed Functions
1. Write a query that selects from the customer_purchases table and numbers each customer’s visits to the farmer’s market (labeling each market date with a different number). Each customer’s first visit is labeled 1, second visit is labeled 2, etc. 

You can either display all rows in the customer_purchases table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits. 

**HINT**: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK().

```
SELECT customer_id, market_date, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date)
AS visit_number	

FROM (
	SELECT DISTINCT customer_id, market_date
	FROM customer_purchases )
AS sub
```

2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, then write another query that uses this one as a subquery (or temp table) and filters the results to only the customer’s most recent visit.

```
SELECT customer_id, market_date, visit_number	

FROM(	
	SELECT customer_id, market_date, 
		ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date DESC) AS visit_number	

	FROM (
		SELECT DISTINCT customer_id, market_date
		FROM customer_purchases )
	AS tmp)
AS visits

WHERE visit_number = 1
```

3. Using a COUNT() window function, include a value along with each row of the customer_purchases table that indicates how many different times that customer has purchased that product_id.
```
SELECT *
	,COUNT(*)  OVER ( PARTITION BY customer_id, product_id) AS count_customer_product_purachases

FROM customer_purchases
```


<div align="center">-</div>

#### String manipulations
1. Some product names in the product table have descriptions like "Jar" or "Organic". These are separated from the product name with a hyphen. Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. Remove any trailing or leading whitespaces. Don't just use a case statement for each product! 

| product_name               | description |
|----------------------------|-------------|
| Habanero Peppers - Organic | Organic     |

**HINT**: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column. 

```
SELECT *, 
	CASE WHEN INSTR(product_name, '-') = 0 THEN NULL
	ELSE TRIM(SUBSTR(product_name, INSTR(product_name, '-') + 1))
	END AS INFO
FROM product
```

2. Filter the query to show any product_size value that contain a number with REGEXP. 

```
SELECT *, 
	CASE WHEN INSTR(product_name, '-') = 0 THEN NULL
	ELSE TRIM(SUBSTR(product_name, INSTR(product_name, '-') + 1))
	END AS INFO
FROM product
WHERE product_size REGEXP '[0-9]'
```

<div align="center">-</div>

#### UNION
1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

**HINT**: There are a possibly a few ways to do this query, but if you're struggling, try the following: 1) Create a CTE/Temp Table to find sales values grouped dates; 2) Create another CTE/Temp table with a rank windowed function on the previous query to create "best day" and "worst day"; 3) Query the second temp table twice, once for the best day, once for the worst day, with a UNION binding them. 

```

WITH
	day_sales AS (
		SELECT *, 
			SUM(quantity * cost_to_customer_per_qty) AS total_customer_sales
		FROM customer_purchases
		GROUP BY market_date),
	day_ranks AS (
		SELECT  *,
			RANK() OVER (ORDER BY total_customer_sales DESC) AS top_rank,
			RANK() OVER (ORDER BY total_customer_sales ASC) AS low_rank
		FROM day_sales)
		
SELECT market_date, total_customer_sales, 'best_day' AS day_rank
FROM day_ranks
WHERE top_rank = 1

UNION

SELECT market_date, total_customer_sales, 'worst_day' AS day_rank
FROM day_ranks
WHERE low_rank = 1
```

***

## Section 3:
You can start this section following *session 5*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question

### Write SQL

#### Cross Join
1. Suppose every vendor in the `vendor_inventory` table had 5 of each of their products to sell to **every** customer on record. How much money would each vendor make per product? Show this by vendor_name and product name, rather than using the IDs.

**HINT**: Be sure you select only relevant columns and rows. Remember, CROSS JOIN will explode your table rows, so CROSS JOIN should likely be a subquery. Think a bit about the row counts: how many distinct vendors, product names are there (x)? How many customers are there (y). Before your final group by you should have the product of those two queries (x\*y). 

```
WITH
	vendor_products AS (
		SELECT DISTINCT vi.vendor_id, v.vendor_name, vi.product_id, p.product_name, vi.original_price
		FROM vendor_inventory AS vi
		JOIN vendor AS v ON vi.vendor_id = v.vendor_id
		JOIN product AS p ON vi.product_id = p.product_id),

	customer_list AS (
		SELECT customer_id
		FROM customer),

	vendor_customer_cross AS (
		SELECT vp.vendor_id, vp.vendor_name, vp.product_id, vp.product_name, vp.original_price, c.customer_id
		FROM vendor_products AS vp
		CROSS JOIN customer_list AS c)

SELECT vendor_name, product_name,
	SUM(original_price * 5) AS total_revenue
FROM vendor_customer_cross	
GROUP BY vendor_name, product_name
```

<div align="center">-</div>

#### INSERT
1. Create a new table "product_units". This table will contain only products where the `product_qty_type = 'unit'`. It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  Name the timestamp column `snapshot_timestamp`.

```
CREATE TABLE	product_units AS 
	SELECT *,
		CURRENT_TIMESTAMP AS snapshot_timestamp
	FROM product
	WHERE product_qty_type = 'unit'
```

2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie). 

```
INSERT INTO product_units VALUES (16, 'Sweet Corn', 'Ear', 1, 'unit', CURRENT_TIMESTAMP)
```

<div align="center">-</div>

#### DELETE 
1. Delete the older record for the whatever product you added.

**HINT**: If you don't specify a WHERE clause, [you are going to have a bad time](https://imgflip.com/i/8iq872).

```
DELETE FROM product_units
WHERE product_id = 16 
	AND snapshot_timestamp = (
		SELECT MIN(snapshot_timestamp)
		FROM product_units
		WHERE product_id = 16)
```

<div align="center">-</div>

#### UPDATE
1. We want to add the current_quantity to the product_units table. First, add a new column, `current_quantity` to the table using the following syntax.
```
ALTER TABLE product_units
ADD current_quantity INT;
```

Then, using `UPDATE`, change the current_quantity equal to the **last** `quantity` value from the vendor_inventory details. 

**HINT**: This one is pretty hard. First, determine how to get the "last" quantity per product. Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) Third, `SET current_quantity = (...your select statement...)`, remembering that WHERE can only accommodate one column. Finally, make sure you have a WHERE statement to update the right row, you'll need to use `product_units.product_id` to refer to the correct row within the product_units table. When you have all of these components, you can run the update statement.

```
UPDATE product_units
SET current_quantity = COALESCE(
	(SELECT vi.quantity
	FROM vendor_inventory AS vi
	WHERE vi.product_id = product_units.product_id
	ORDER BY vi.market_date DESC
	LIMIT 1),
	0)
WHERE product_units.product_id IS NOT NULL
```


*** 

## Section 4:
You can start this section anytime.

Steps to complete this part of the assignment:
- Read the article
- Write, within this markdown file, <1000 words.

### Ethics

Read: Boykis, V. (2019, October 16). _Neural nets are just people all the way down._ Normcore Tech. <br>
    https://vicki.substack.com/p/neural-nets-are-just-people-all-the

**What are the ethical issues important to this story?**

Consider, for example, concepts of labour, bias, LLM proliferation, moderating content, intersection of technology and society, ect. 


```
- The annotation work is often geographically outsourced to the global south, frequently with low wages, minimal labour protections, and limited transparency. This labour exploitation is not widely known by the users of LLMs. The involvment of humans in developing these training sets is not widely known by users of LLMs allowing for the proliferation of misconceptions about how the AIs actually work. Annotations done by humans, especially across large and diverse groups, will have inconsistent annotations.

- Biases in labour pools produce culturally skewed understandings of the data which infuses cultural assumptions, stereotypes, and biases that shape the dataset labels (such as with ImageNet). Similar biases in the provided training data will also amplify issues that are inherent to the datasets such as racial discrimination. These datasets are also often biased by under-represention of marginalized communities.

- The data that these models are trained on often taken from sources in a way that is ethically and legally questionable. 

- The rapid proliferation of LLMs in public use makes moderation of it's output particularly difficult. The safety filters usually have many blind spots and it is very difficult to train models to automatically filter harmful content since it does not have an inherent understanding of what is harmful to humans. Additionally, training these safety filters introduces new biases that can have unintended impacts on its output, such as creating political biases. The ease of use of these models by the public also provides a very powerful and under-regulated tool for individuals who would like to deliberately promote false, misleading, and harmful content.



```
