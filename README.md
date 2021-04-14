# SQL
-- 1) In terms of both $ revenue and volume (# of transactions), who were the top 5 accounts and their associated transaction $ amount/volume during the time period given?

SELECT
   ROUND(SUM(price), 2) AS total_price,
   account_id,
   COUNT(letter_id) AS letter_count 
FROM
   (
      SELECT DISTINCT
         account_id,
         letter_id,
         price,
         delivered,
         pages,
      FROM
         Terminal_data.raw_data 
   )
GROUP BY
   account_id 
ORDER BY
   total_price DESC LIMIT 5 ;

-- 2) How many letters were sent as multi-page, double-sided, color?

SELECT DISTINCT
   COUNT(letter_id) AS letter_count 
FROM
   (
      SELECT DISTINCT
         letter_id,
         pages,
         double_sided,
         color,
      FROM
         Terminal_data.raw_data 
      WHERE
         pages > 1 
         AND double_sided = TRUE 
         AND color = TRUE 
   );

     -- What percentage of letters were delivered? For delivered letters, what was the median
  -- delivery time and average delivery time?

SELECT DISTINCT
   COUNT(letter_id) AS letter_count_delivered,
   ROUND(COUNT(letter_id) * 100.0 / ( 
   SELECT
      COUNT(*) 
   FROM
      Terminal_data.raw_data), 2) AS perc_letter_delivered 
   FROM
      Terminal_data.raw_data 
   WHERE
      delivered = TRUE ;

      
SELECT
   PERCENTILE_CONT(day_diff, 0.5) OVER() AS median_time 
FROM
   (
      SELECT DISTINCT
         letter_id,
         date_ordered,
         scan_date,
         DATE_DIFF(scan_date, date_ordered, DAY) AS day_diff 
      FROM
         Terminal_data.raw_data 
      WHERE
         delivered = TRUE 
      ORDER BY
         DATE_DIFF(scan_date, date_ordered, DAY) ASC 
   )
   LIMIT 1 ;


SELECT
   ROUND(AVG(day_diff)) 
FROM
   (
      SELECT DISTINCT
         letter_id,
         date_ordered,
         scan_date,
         delivered,
         DATE_DIFF(scan_date, date_ordered, DAY) AS day_diff 
      FROM
         Terminal_data.raw_data 
      WHERE
         delivered = TRUE 
   );

   -- What was the minimum day of week $ revenue for July 2015?


(
SELECT
   MIN(price_total) AS minimum_revenue, week_day 
FROM
   (
      SELECT
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered)AS day_of_week,
         ROUND(SUM(price), 2) AS price_total,
         CASE
            WHEN
               EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 1 
      THEN
         'Sunday' 
      WHEN
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 2 
      THEN
         'Monday' 
      WHEN
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 3 
      THEN
         'Tuesday' 
      WHEN
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 4 
      THEN
         'Wednesday' 
      WHEN
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 5 
      THEN
         'Thursday' 
      WHEN
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 6 
      THEN
         'Friday' 
      WHEN
         EXTRACT(DAYOFWEEK 
      FROM
         date_ordered) = 7 
      THEN
         'Saturday' 
      ELSE
         'Invalid_day' 
         END
         AS week_day, 
      FROM
         Terminal_data.raw_data 
      WHERE
         date_ordered BETWEEN '2015-07-01' AND '2015-07-31' 
      GROUP BY
         date_ordered 
      ORDER BY
         day_of_week 
   )
GROUP BY
   week_day 
ORDER BY
   minimum_revenue ASC LIMIT 1 )

;

-- What is the average sheet count of all letters? (2 page double-sided letter = 1 sheet)(if its just one page, then it's half a sheet)

SELECT
   ROUND(AVG(sheet_count)) AS avg_sheet_count 
FROM
   (
      SELECT DISTINCT
         letter_id,
         pages,
         double_sided,
         pages / 2 AS sheet_count 
      FROM
         Terminal_data.raw_data 
   );






