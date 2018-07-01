# SQL-HW

-- 1A. DISPLAY THE FIRST AND LAST NAMES OF ALL ACTORS FROM THE TABLE `ACTOR`

USE SAKILA;

SELECT FIRST_NAME,LAST_NAME 
FROM ACTOR;

-- 1B. DISPLAY THE FIRST AND LAST NAME OF EACH ACTOR IN A SINGLE COLUMN IN UPPER CASE LETTERS. 
-- NAME THE COLUMN `ACTOR NAME`. 

SELECT UPPER(CONCAT(FIRST_NAME,' ',LAST_NAME)) AS `ACTOR NAME`
FROM ACTOR;

-- 2A. YOU NEED TO FIND THE ID NUMBER, FIRST NAME, AND LAST NAME OF AN ACTOR, 
-- OF WHOM YOU KNOW ONLY THE FIRST NAME, "JOE." 
-- WHAT IS ONE QUERY WOULD YOU USE TO OBTAIN THIS INFORMATION?	

SELECT ACTOR_ID, FIRST_NAME, LAST_NAME
FROM ACTOR
WHERE FIRST_NAME = 'JOE';


-- 2B. FIND ALL ACTORS WHOSE LAST NAME CONTAIN THE LETTERS `GEN`:

SELECT ACTOR_ID, FIRST_NAME, LAST_NAME
FROM ACTOR
WHERE LAST_NAME LIKE '%GEN%';


-- 2C. FIND ALL ACTORS WHOSE LAST NAMES CONTAIN THE LETTERS `LI`. 
-- THIS TIME, ORDER THE ROWS BY LAST NAME AND FIRST NAME, IN THAT ORDER:

SELECT ACTOR_ID, FIRST_NAME, LAST_NAME
FROM ACTOR
WHERE LAST_NAME LIKE '%LI%'
ORDER BY LAST_NAME,FIRST_NAME;


-- 2D. USING `IN`, DISPLAY THE `COUNTRY_ID` AND `COUNTRY` COLUMNS OF THE FOLLOWING COUNTRIES: 
-- AFGHANISTAN, BANGLADESH, AND CHINA:


SELECT COUNTRY_ID, COUNTRY
FROM COUNTRY
WHERE COUNTRY IN ('AFGHANISTAN', 'BANGLADESH','CHINA');

-- 3A. ADD A `MIDDLE_NAME` COLUMN TO THE TABLE `ACTOR`. 
-- POSITION IT BETWEEN `FIRST_NAME` AND `LAST_NAME`. 
-- HINT: YOU WILL NEED TO SPECIFY THE DATA TYPE

ALTER TABLE ACTOR 
ADD COLUMN MIDDLE_NAME VARCHAR(45) AFTER FIRST_NAME;


-- 3B. YOU REALIZE THAT SOME OF THESE ACTORS HAVE TREMENDOUSLY LONG LAST NAMES. 
-- CHANGE THE DATA TYPE OF THE `MIDDLE_NAME` COLUMN TO `BLOBS`.

ALTER TABLE ACTOR MODIFY MIDDLE_NAME BLOB;


-- 3C. NOW DELETE THE `MIDDLE_NAME` COLUMN.

ALTER TABLE ACTOR DROP COLUMN MIDDLE_NAME;

-- 4A. LIST THE LAST NAMES OF ACTORS, AS WELL AS HOW MANY ACTORS HAVE THAT LAST NAME.

SELECT LAST_NAME, COUNT(ACTOR_ID) AS FREQ
FROM ACTOR
GROUP BY LAST_NAME;


-- 4B. LIST LAST NAMES OF ACTORS AND THE NUMBER OF ACTORS WHO HAVE THAT LAST NAME, 
-- BUT ONLY FOR NAMES THAT ARE SHARED BY AT LEAST TWO ACTORS


SELECT LAST_NAME, COUNT(ACTOR_ID) AS FREQ
FROM ACTOR
GROUP BY LAST_NAME
HAVING FREQ > 2;
  	
-- 4C. OH, NO! THE ACTOR `HARPO WILLIAMS` WAS ACCIDENTALLY ENTERED IN THE `ACTOR` 
-- TABLE AS `GROUCHO WILLIAMS`, THE NAME OF HARPO'S SECOND COUSIN'S HUSBAND'S YOGA TEACHER. 
-- WRITE A QUERY TO FIX THE RECORD.

UPDATE ACTOR
SET  FIRST_NAME = 'HARPO'
WHERE FIRST_NAME = 'GROUCHO'  AND LAST_NAME = 'WILLIAMS';

-- 4D. PERHAPS WE WERE TOO HASTY IN CHANGING `GROUCHO` TO `HARPO`. 
-- IT TURNS OUT THAT `GROUCHO` WAS THE CORRECT NAME AFTER ALL! IN A SINGLE QUERY, 
-- IF THE FIRST NAME OF THE ACTOR IS CURRENTLY `HARPO`, CHANGE IT TO `GROUCHO`. 
-- OTHERWISE, CHANGE THE FIRST NAME TO `MUCHO GROUCHO`, AS THAT IS EXACTLY 
-- WHAT THE ACTOR WILL BE WITH THE GRIEVOUS ERROR. 
-- BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO `MUCHO GROUCHO`, 
-- HOWEVER! (HINT: UPDATE THE RECORD USING A UNIQUE IDENTIFIER.)

UPDATE ACTOR 
SET FIRST_NAME= 'HARPO'
WHERE FIRST_NAME='GROUCHO' AND LAST_NAME='WILLIAMS';

-- 5A. YOU CANNOT LOCATE THE SCHEMA OF THE `ADDRESS` TABLE. 
-- WHICH QUERY WOULD YOU USE TO RE-CREATE IT?

-- FOR FORMATTED OUTPUT
DESCRIBE ADDRESS;

-- SQL STATEMENT THAT CAN BE USED TO CREATE A TABLE
SHOW CREATE TABLE  ADDRESS;

-- 6A. USE `JOIN` TO DISPLAY THE FIRST AND LAST NAMES, AS WELL AS THE ADDRESS, OF EACH STAFF MEMBER. 
-- USE THE TABLES `STAFF` AND `ADDRESS`:


SELECT S.FIRST_NAME, S.LAST_NAME, A.ADDRESS
FROM STAFF S
INNER JOIN ADDRESS  A
ON S.ADDRESS_ID = A.ADDRESS_ID;

-- 6B. USE `JOIN` TO DISPLAY THE TOTAL AMOUNT RUNG UP BY EACH STAFF MEMBER IN AUGUST OF 2005. 
-- USE TABLES `STAFF` AND `PAYMENT`. 
  	
SELECT STABLE.FIRST_NAME,STABLE.LAST_NAME,SUMMARY.TOTAL_PAYMENTS
FROM STAFF as STABLE
INNER JOIN 
    (SELECT S.STAFF_ID, SUM(P.AMOUNT) AS TOTAL_PAYMENTS
    FROM STAFF AS S
    INNER JOIN PAYMENT P
    ON S.STAFF_ID = P.STAFF_ID
    WHERE P.PAYMENT_DATE >='2005-08-01 00:00:00'
    AND P.PAYMENT_DATE <'2005-09-01 00:00:00'
    GROUP BY S.STAFF_ID) as SUMMARY
ON STABLE.STAFF_ID = SUMMARY.STAFF_ID;

-- 6C. LIST EACH FILM AND THE NUMBER OF ACTORS WHO ARE LISTED FOR THAT FILM. 
-- USE TABLES `FILM_ACTOR` AND `FILM`. USE INNER JOIN.

SELECT F.TITLE, COUNT(FA.ACTOR_ID) AS NUMBER_OF_ACTORS
FROM FILM F
INNER JOIN FILM_ACTOR FA
ON F.FILM_ID = FA.FILM_ID
GROUP BY F.TITLE;

-- 6D. HOW MANY COPIES OF THE FILM `HUNCHBACK IMPOSSIBLE` EXIST IN THE INVENTORY SYSTEM?

SELECT COUNT(INVENTORY_ID) AS NUMBER_OF_COPIES
FROM INVENTORY 
WHERE FILM_ID IN (SELECT FILM_ID FROM FILM 
WHERE TITLE = 'HUNCHBACK IMPOSSIBLE');

-- 6E. USING THE TABLES `PAYMENT` AND `CUSTOMER` AND THE `JOIN` COMMAND, 
-- LIST THE TOTAL PAID BY EACH CUSTOMER. LIST THE CUSTOMERS ALPHABETICALLY BY LAST NAME:

SELECT CUS.FIRST_NAME, CUS.LAST_NAME,SUMMARY.TOTAL AS TOTAL_AMOUNT_PAID
FROM CUSTOMER CUS
INNER JOIN
    (SELECT C.CUSTOMER_ID, SUM(P.AMOUNT) TOTAL
    FROM PAYMENT P
    INNER JOIN CUSTOMER C
    ON P.CUSTOMER_ID = C.CUSTOMER_ID
    GROUP BY C.CUSTOMER_ID) SUMMARY
ON SUMMARY.CUSTOMER_ID = CUS.CUSTOMER_ID
ORDER BY TOTAL_AMOUNT_PAID DESC;

-- 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. 
-- As an unintended consequence, films starting with the letters `K` and `Q` have also soared in popularity. 
-- Use subqueries to display the titles of movies starting with the letters `K` and `Q` 
-- whose language is English. 

SELECT TITLE 
FROM FILM 
WHERE (TITLE LIKE 'K%' OR TITLE LIKE 'Q%') AND LANGUAGE_ID  
IN(SELECT LANGUAGE_ID FROM LANGUAGE
WHERE NAME='ENGLISH');

-- 7b. Use subqueries to display all actors who appear in the film `Alone Trip`.
   
SELECT ACTOR.ACTOR_ID, ACTOR.FIRST_NAME, ACTOR.LAST_NAME
FROM ACTOR
INNER JOIN           
    (SELECT ACTOR_ID 
     FROM FILM_ACTOR
     WHERE FILM_ID IN 
        (SELECT FILM_ID 
         FROM FILM
         WHERE TITLE = 'ALONE TRIP')) AS TEMP
ON TEMP.ACTOR_ID = ACTOR.ACTOR_ID;


-- 7c. You want to run an email marketing campaign in Canada, 
-- for which you will need the names and email addresses of all Canadian customers. 
-- Use joins to retrieve this information.

SELECT FIRST_NAME, LAST_NAME, EMAIL 
FROM CUSTOMER 
WHERE ADDRESS_ID IN (
    SELECT ADDRESS_ID 
    FROM ADDRESS
    WHERE CITY_ID IN 
        (
        SELECT CITY_ID 
        FROM CITY
        WHERE COUNTRY_ID IN 
            (
            SELECT COUNTRY_ID 
            FROM COUNTRY
            WHERE COUNTRY ='CANADA')));

-- 7d. Sales have been lagging among young families, and you wish to 
-- target all family movies for a promotion. Identify all movies categorized as famiy films.

SELECT TITLE 
FROM FILM
WHERE FILM_ID IN (
    SELECT FILM_ID 
    FROM FILM_CATEGORY
    WHERE CATEGORY_ID IN 
        (
         SELECT CATEGORY_ID 
         FROM  CATEGORY
         WHERE NAME = 'FAMILY'));

-- 7e. Display the most frequently rented movies in descending order.
 
SELECT TITLE, SUM(TOTAL_RENTALS) AS TOTAL_RENTALS
FROM 
(SELECT Q.TITLE, TABLE2.FILM_ID, TABLE2.TOTAL_RENTALS AS TOTAL_RENTALS
    FROM FILM Q
    INNER JOIN
        (SELECT I. FILM_ID, TABLE1.INVENTORY_ID AS INVENTORY_ID,TABLE1.TOTAL_RENTALS AS TOTAL_RENTALS
        FROM INVENTORY I
        INNER JOIN
            (SELECT INVENTORY_ID, COUNT(RENTAL_ID) AS TOTAL_RENTALS
            FROM RENTAL 
            GROUP BY INVENTORY_ID) AS TABLE1
        ON I. INVENTORY_ID = TABLE1.INVENTORY_ID) AS TABLE2
    ON Q.FILM_ID = TABLE2.FILM_ID) AS TABLE3
GROUP BY TITLE
ORDER BY TOTAL_RENTALS DESC;


-- 7f. Write a query to display how much business, in dollars, each store brought in.


SELECT A.ADDRESS AS STORE_ADDRESS, TABLE2.TOTAL AS TOTAL_AMOUNT
FROM ADDRESS A
INNER JOIN
    (SELECT STORE.ADDRESS_ID, TABLE1.TOTAL AS TOTAL
     FROM STORE
     INNER JOIN
        (SELECT STAFF_ID, SUM(AMOUNT) AS TOTAL
         FROM PAYMENT
         GROUP BY STAFF_ID) TABLE1
    ON STORE.MANAGER_STAFF_ID=TABLE1.STAFF_ID) TABLE2
ON A.ADDRESS_ID = TABLE2.ADDRESS_ID;


-- 7g. Write a query to display for each store its store ID, city, and country.

SELECT TABLE3.STORE_ID, TABLE3.CITY,  CO.COUNTRY
FROM COUNTRY CO
INNER JOIN
    (SELECT TABLE2.STORE_ID, TABLE2.CITY_ID, C.CITY, C.COUNTRY_ID
    FROM CITY C
    INNER JOIN
        (SELECT  TABLE1.STORE_ID, A.CITY_ID
        FROM    ADDRESS A
        INNER JOIN
            (SELECT STORE_ID, ADDRESS_ID 
            FROM STORE) TABLE1
        ON TABLE1.ADDRESS_ID = A.ADDRESS_ID) TABLE2
    ON TABLE2.CITY_ID = C.CITY_ID)TABLE3
ON CO.COUNTRY_ID = TABLE3.COUNTRY_ID;


-- 7h. List the top five genres in gross revenue in descending order. 
-- (**Hint**: you may need to use the following tables: category, film_category, inventory, payment, and rental.)

SELECT C.NAME,SUM(TABLE4.TOTAL) AS TOTAL
FROM CATEGORY C
INNER JOIN
    (SELECT F.CATEGORY_ID, TABLE3.FILM_ID, TABLE3.INVENTORY_ID, TABLE3.RENTAL_ID, TABLE3.TOTAL
    FROM  FILM_CATEGORY F
    INNER JOIN
        (SELECT I.FILM_ID, TABLE2.INVENTORY_ID, TABLE2.RENTAL_ID, TABLE2.TOTAL
        FROM INVENTORY I
        INNER JOIN
            (SELECT R.INVENTORY_ID, TABLE1.RENTAL_ID, TABLE1.TOTAL
            FROM  RENTAL R
            INNER JOIN
                (SELECT RENTAL_ID,SUM(AMOUNT) AS TOTAL
                FROM PAYMENT
                GROUP BY RENTAL_ID) TABLE1
            ON TABLE1.RENTAL_ID = R.RENTAL_ID) TABLE2
        ON TABLE2.INVENTORY_ID = I.INVENTORY_ID) TABLE3
    ON TABLE3.FILM_ID = F.FILM_ID) TABLE4
ON TABLE4.CATEGORY_ID = C.CATEGORY_ID
GROUP BY C.NAME
ORDER BY TOTAL DESC
LIMIT 5;

-- 8a. In your new role as an executive, you would like to have an easy way of 
-- viewing the Top five genres by gross revenue. Use the solution from the problem 
-- above to create a view. If you haven't solved 7h, you can substitute another query to create a view.

CREATE VIEW top_five_genres AS
SELECT C.NAME,SUM(TABLE4.TOTAL) AS TOTAL
FROM CATEGORY C
INNER JOIN
    (SELECT F.CATEGORY_ID, TABLE3.FILM_ID, TABLE3.INVENTORY_ID, TABLE3.RENTAL_ID, TABLE3.TOTAL
    FROM  FILM_CATEGORY F
    INNER JOIN
        (SELECT I.FILM_ID, TABLE2.INVENTORY_ID, TABLE2.RENTAL_ID, TABLE2.TOTAL
        FROM INVENTORY I
        INNER JOIN
            (SELECT R.INVENTORY_ID, TABLE1.RENTAL_ID, TABLE1.TOTAL
            FROM  RENTAL R
            INNER JOIN
                (SELECT RENTAL_ID,SUM(AMOUNT) AS TOTAL
                FROM PAYMENT
                GROUP BY RENTAL_ID) TABLE1
            ON TABLE1.RENTAL_ID = R.RENTAL_ID) TABLE2
        ON TABLE2.INVENTORY_ID = I.INVENTORY_ID) TABLE3
    ON TABLE3.FILM_ID = F.FILM_ID) TABLE4
ON TABLE4.CATEGORY_ID = C.CATEGORY_ID
GROUP BY C.NAME
ORDER BY TOTAL DESC
LIMIT 5;


-- 8b. How would you display the view that you created in 8a?
SELECT * FROM top_five_genres;

-- 8c. You find that you no longer need the view `top_five_genres`. Write a query to delete it.
DROP VIEW top_five_genres;
