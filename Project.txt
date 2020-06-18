/* QUERY 1, used for the first slide. I also used EXCEL to count number of titles per category to build a chart. */

WITH t1 AS (
  SELECT fi.title film_title, ca.name category_name, re.rental_id rental_id
    FROM category ca
    JOIN film_category fica
    ON ca.category_id = fica.category_id
    JOIN film fi
    ON fi.film_id = fica.film_id
    JOIN inventory inv
    ON fi.film_id = inv.film_id
    JOIN rental re
    ON inv.inventory_id = re.inventory_id
    WHERE ca.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
)

SELECT DISTINCT(film_title), category_name,
      COUNT(film_title) OVER (PARTITION BY film_title) rental_count
  FROM t1
ORDER BY 2, 1;

-----------------------------
/* Query 2, used for the second slide. Again, I also used COUNT clause and Excel to count how many movies in each quarter. */

SELECT fi.title title, ca.name category, fi.rental_duration rental_duration,
        NTILE(4) OVER (ORDER BY rental_duration) standard_quartile
  FROM category ca
JOIN film_category fica
ON ca.category_id = fica.category_id
JOIN film fi
ON fi.film_id = fica.film_id
WHERE ca.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')

--------------------------------
/* Query 3, used for the third slide. I could have used a WINDOW Function on the COUNT clause. I used CASE clause to rename the standard quartile to make the chart easier to follow.*/

WITH t1 AS (
SELECT fi.title title, ca.name category, fi.rental_duration rental_duration,
        NTILE(4) OVER (ORDER BY rental_duration) standard_quartile
  FROM category ca
JOIN film_category fica
ON ca.category_id = fica.category_id
JOIN film fi
ON fi.film_id = fica.film_id
WHERE ca.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
)
SELECT DISTINCT category, standard_quartile,
                COUNT(*) counts
  FROM t1
GROUP BY 1, 2
ORDER BY 1,2

-------------------------------
/*Query 4, used for the last slide.*/

WITH t1 AS (
  SELECT fi.title film_title, ca.name category_name, re.rental_id rental_id, pa.amount amount
    FROM category ca
    JOIN film_category fica
    ON ca.category_id = fica.category_id
    JOIN film fi
    ON fi.film_id = fica.film_id
    JOIN inventory inv
    ON fi.film_id = inv.film_id
    JOIN rental re
    ON inv.inventory_id = re.inventory_id
  	LEFT JOIN payment pa
  	ON pa.rental_id = re.rental_id
    WHERE ca.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
)
SELECT DISTINCT category_name,
      COUNT(film_title) OVER (PARTITION BY category_name) rental_count,
      SUM(amount) OVER (PARTITION BY category_name) rental_total
  FROM t1
ORDER BY 1
