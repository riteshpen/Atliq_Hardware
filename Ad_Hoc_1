# 1
SELECT market FROM dim_customer
WHERE region = 'APAC' and customer = 'Atliq Exclusive';

#2
WITH yearly_counts AS (
    SELECT fiscal_year, COUNT(DISTINCT product_code) AS unique_products
    FROM fact_sales_monthly
    GROUP BY fiscal_year
),
calculate_change AS (
    SELECT MAX(CASE WHEN fiscal_year = 2020 THEN unique_products ELSE NULL END) AS unique_products_2020,
        MAX(CASE WHEN fiscal_year = 2021 THEN unique_products ELSE NULL END) AS unique_products_2021
    FROM yearly_counts
)
SELECT unique_products_2020, unique_products_2021, 
round(((unique_products_2021 - unique_products_2020) / unique_products_2020) * 100, 2) AS percentage_chg
FROM calculate_change;

#3
SELECT segment, count(distinct product_code) as product_counts FROM dim_product
GROUP BY segment
ORDER BY product_counts DESC;

#4
WITH yearly_counts AS (
    SELECT dp.segment,
        COUNT(DISTINCT CASE WHEN fs.fiscal_year = 2020 THEN dp.product_code END) AS product_count_2020,
        COUNT(DISTINCT CASE WHEN fs.fiscal_year = 2021 THEN dp.product_code END) AS product_count_2021
    FROM dim_product dp
    LEFT JOIN fact_sales_monthly fs ON dp.product_code = fs.product_code
    WHERE fs.fiscal_year IN (2020, 2021)
    GROUP BY dp.segment
)
SELECT segment, product_count_2020, product_count_2021, product_count_2021 - product_count_2020 as difference FROM yearly_counts
ORDER BY difference DESC

#5
WITH MinMaxCosts as (
	SELECT MIN(manufacturing_cost) as min_cost, MAX(manufacturing_cost) as max_cost 
    FROM fact_manufacturing_cost
)

SELECT man.product_code, pro.product, man.manufacturing_cost FROM fact_manufacturing_cost man
LEFT JOIN dim_product pro ON pro.product_code = man.product_code
JOIN MinMaxCosts mmc ON man.manufacturing_cost = mmc.min_cost OR man.manufacturing_cost = mmc.max_cost;

#6
SELECT pre.customer_code, cust.customer, avg(pre.pre_invoice_discount_pct) as average_discount_percentage 
FROM fact_pre_invoice_deductions pre
LEFT JOIN dim_customer cust ON cust.customer_code = pre.customer_code
WHERE fiscal_year = 2021
GROUP BY pre.customer_code, cust.customer
ORDER BY average_discount_percentage DESC LIMIT 5

#7
SELECT MONTH(sale.date) AS sale_month, YEAR(sale.date) AS sale_year, SUM(sale.sold_quantity) AS sale_gross_sales_amount
FROM fact_sales_monthly sale
LEFT JOIN  dim_customer cust ON sale.customer_code = cust.customer_code
WHERE cust.customer = 'Atliq Exclusive'
GROUP BY MONTH(sale.date), YEAR(sale.date)
ORDER BY sale_year, sale_month;

#8
SELECT SUM(sold_quantity) AS total_sold_quantity,
    CASE 
        WHEN MONTH(date) BETWEEN 1 AND 3 THEN 1
        WHEN MONTH(date) BETWEEN 4 AND 6 THEN 2
        WHEN MONTH(date) BETWEEN 7 AND 9 THEN 3
        ELSE 4
    END AS quarter
FROM fact_sales_monthly
GROUP BY quarter
ORDER BY quarter, total_sold_quantity DESC;

#9
WITH cte AS (SELECT dc.channel, SUM(fsm.sold_quantity * fgp.gross_price) AS gross_sales
  FROM fact_sales_monthly fsm
  JOIN fact_gross_price fgp ON fsm.product_code = fgp.product_code AND fsm.fiscal_year = fgp.fiscal_year
  JOIN dim_customer dc ON fsm.customer_code = dc.customer_code
  WHERE fsm.fiscal_year = 2021
  GROUP BY dc.channel
)
SELECT channel, ROUND(gross_sales / 1000000, 2) AS gross_sales_mln, ROUND(100.0 * gross_sales / (SELECT SUM(gross_sales) FROM cte), 2) AS percentage
FROM cte
ORDER BY gross_sales DESC;

#10
CREATE TEMPORARY TABLE temp_sales
SELECT product_code, SUM(sold_quantity) AS total_sold_quantity
FROM fact_sales_monthly
WHERE fiscal_year = 2021
GROUP BY product_code;

SELECT dp.division, dp.product_code, dp.product, ts.total_sold_quantity,
  DENSE_RANK() OVER (PARTITION BY dp.division ORDER BY ts.total_sold_quantity DESC) AS rank_order
FROM dim_product dp
JOIN temp_sales ts ON dp.product_code = ts.product_code
ORDER BY dp.division, rank_order;

SELECT division, product_code, product, total_sold_quantity, rank_order
FROM (SELECT dp.division, dp.product_code, dp.product, ts.total_sold_quantity,
      DENSE_RANK() OVER (PARTITION BY dp.division ORDER BY ts.total_sold_quantity DESC) AS rank_order
FROM dim_product dp
JOIN temp_sales ts ON dp.product_code = ts.product_code
  ) ranked_products
WHERE rank_order <= 3
ORDER BY division, rank_order;

DROP TEMPORARY TABLE temp_sales;


