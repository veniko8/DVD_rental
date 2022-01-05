#SLIDE1


WITH sub AS 
  (WITH top10 AS (SELECT CONCAT(c.first_name,' ', c.last_name) customer, c.customer_id id, SUM(p.amount)
    FROM customer c
    JOIN payment p
    ON p.customer_id=c.customer_id
    GROUP BY 1,2
    ORDER BY 3 desc
    LIMIT 10)

     SELECT DATE_TRUNC('month',p.payment_date) date, top10.customer customer, COUNT(*), SUM(p.amount) sum_paid
     FROM top10
     JOIN payment p
     ON top10.id=p.customer_id
     GROUP BY 1,2
     ORDER BY 2)

SELECT CONCAT(DATE_PART('year',sub.date),'-',DATE_PART('month',sub.date)),sub.customer,sub.sum_paid, sum_paid - LAG(sum_paid) OVER (PARTITION BY customer ORDER BY customer) AS diff
FROM sub;

#SLIDE2


WITH rentalcount AS (WITH sub AS (SELECT c.name category, f.title title, f.film_id id
            FROM category c
            JOIN film_category fc
            ON c.category_id=fc.category_id
            JOIN film f
            ON fc.film_id=f.film_id
            WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music'))

  SELECT sub.title, sub.category, COUNT( rental_id) rental_count
  FROM sub
  JOIN inventory
  ON sub.id =inventory.film_id
  JOIN rental
  ON rental.inventory_id=inventory.inventory_id
  GROUP BY 1,2
  ORDER BY 2 ,1)
  
  
SELECT DISTINCT rentalcount.category, SUM(rentalcount.rental_count) OVER (PARTITION BY rentalcount.category ORDER BY rentalcount.category) 
FROM rentalcount
ORDER BY sum desc;



#SLIDE3

WITH family_movies AS (SELECT c.name category, f.title title, f.film_id id
            FROM category c
            JOIN film_category fc
            ON c.category_id=fc.category_id
            JOIN film f
            ON fc.film_id=f.film_id
            WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music'))
            
SELECT COUNT(family_movies.id), date_trunc('week', rental_date) rental_week
FROM family_movies
JOIN inventory i
ON family_movies.id=i.film_id
JOIN rental 
ON i.inventory_id = rental.inventory_id
GROUP BY rental_week
HAVING date_trunc('week', rental_date) < '2006-01-01'
ORDER BY rental_week;



 
#SLIDE4



WITH top_countries as (
    SELECT SUM(p.amount),country.country, min(country.country_id) as country_id
    FROM country 
    JOIN city ON country.country_id = city.country_id
    JOIN address a ON a.city_id = city.city_id
    JOIN customer cust ON cust.address_id = a.address_id
    JOIN payment p ON p.customer_id = cust.customer_id
    GROUP BY 2
    ORDER BY 1 DESC
    LIMIT 5),

  count_category_per_top_country as
  (SELECT top_countries.country, cat.name as category_name , COUNT(cat.name) as total_films_in_category,
  NTILE(5) OVER(PARTITION BY top_countries.country ORDER BY COUNT(cat.name)) AS femtile
  FROM top_countries
  JOIN city ON top_countries.country_id = city.country_id
  JOIN address a ON a.city_id = city.city_id
  JOIN customer cust ON cust.address_id = a.address_id
  JOIN payment p ON p.customer_id = cust.customer_id
  JOIN rental r ON r.rental_id = p.rental_id
  JOIN inventory i ON i.inventory_id = r.inventory_id
  JOIN film f ON i.film_id = f.film_id
  JOIN film_category fcat ON f.film_id = fcat.film_id
  JOIN category cat ON fcat.category_id = cat.category_id
  GROUP BY 1,2)

SELECT ccptc.country, ccptc.category_name, ccptc.total_films_in_category
FROM count_category_per_top_country ccptc
WHERE ccptc.femtile = 5;
