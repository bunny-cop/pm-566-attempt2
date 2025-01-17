Lab 10
================
Xiaofan Zhu
2022-11-03

``` r
if (!require(RSQLite)) install.packages(c("RSQLite"))
```

    ## Loading required package: RSQLite

``` r
if (!require(DBI))     install.packages(c("DBI"))
```

    ## Loading required package: DBI

``` r
library(RSQLite)
library(DBI)
```

# Data setup

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")
# Download tables
actor <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/actor.csv")
rental <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/rental.csv")
customer <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/customer.csv")
payment <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/payment_p2007_01.csv")
# Copy data.frames to database
dbWriteTable(con, "actor", actor)
dbWriteTable(con, "rental", rental)
dbWriteTable(con, "customer", customer)
dbWriteTable(con, "payment", payment)
dbListTables(con)
```

    ## [1] "actor"    "customer" "payment"  "rental"

``` sql
PRAGMA table_info(actor)
```

``` r
x1
```

    ##   cid        name    type notnull dflt_value pk
    ## 1   0    actor_id INTEGER       0         NA  0
    ## 2   1  first_name    TEXT       0         NA  0
    ## 3   2   last_name    TEXT       0         NA  0
    ## 4   3 last_update    TEXT       0         NA  0

``` sql
PRAGMA table_info(actor)
```

| cid | name        | type    | notnull | dflt_value |  pk |
|:----|:------------|:--------|--------:|:-----------|----:|
| 0   | actor_id    | INTEGER |       0 | NA         |   0 |
| 1   | first_name  | TEXT    |       0 | NA         |   0 |
| 2   | last_name   | TEXT    |       0 | NA         |   0 |
| 3   | last_update | TEXT    |       0 | NA         |   0 |

4 records

``` r
dbGetQuery(con,"
      PRAGMA table_info(actor)
           "
)
```

    ##   cid        name    type notnull dflt_value pk
    ## 1   0    actor_id INTEGER       0         NA  0
    ## 2   1  first_name    TEXT       0         NA  0
    ## 3   2   last_name    TEXT       0         NA  0
    ## 4   3 last_update    TEXT       0         NA  0

# Exercise 1

Retrieve the actor ID, first name and last name for all actors using the
actor table. Sort by last name and then by first name.

``` r
dbGetQuery(con,"
SELECT   actor_id, first_name, last_name
FROM    actor
ORDER by last_name, first_name
LIMIT 15
           ")
```

    ##    actor_id first_name last_name
    ## 1        58  CHRISTIAN    AKROYD
    ## 2       182     DEBBIE    AKROYD
    ## 3        92    KIRSTEN    AKROYD
    ## 4       118       CUBA     ALLEN
    ## 5       145        KIM     ALLEN
    ## 6       194      MERYL     ALLEN
    ## 7        76   ANGELINA   ASTAIRE
    ## 8       112    RUSSELL    BACALL
    ## 9       190     AUDREY    BAILEY
    ## 10       67    JESSICA    BAILEY
    ## 11      115   HARRISON      BALE
    ## 12      187      RENEE      BALL
    ## 13       47      JULIA BARRYMORE
    ## 14      158     VIVIEN  BASINGER
    ## 15      174    MICHAEL    BENING

Try in SQL directly:

``` sql
SELECT   actor_id, first_name, last_name
FROM    actor
ORDER by last_name, first_name
```

| actor_id | first_name | last_name |
|---------:|:-----------|:----------|
|       58 | CHRISTIAN  | AKROYD    |
|      182 | DEBBIE     | AKROYD    |
|       92 | KIRSTEN    | AKROYD    |
|      118 | CUBA       | ALLEN     |
|      145 | KIM        | ALLEN     |
|      194 | MERYL      | ALLEN     |
|       76 | ANGELINA   | ASTAIRE   |
|      112 | RUSSELL    | BACALL    |
|      190 | AUDREY     | BAILEY    |
|       67 | JESSICA    | BAILEY    |

Displaying records 1 - 10

# Exercise 2

Retrive the actor ID, first name, and last name for actors whose last
name equals ‘WILLIAMS’ or ‘DAVIS’.

``` r
dbGetQuery(con,"
SELECT   actor_id, first_name, last_name
FROM    actor
WHERE last_name IN ('WILLIAMS', 'DAVIS')
ORDER BY last_name
")
```

    ##   actor_id first_name last_name
    ## 1        4   JENNIFER     DAVIS
    ## 2      101      SUSAN     DAVIS
    ## 3      110      SUSAN     DAVIS
    ## 4       72       SEAN  WILLIAMS
    ## 5      137     MORGAN  WILLIAMS
    ## 6      172    GROUCHO  WILLIAMS

# Exercise 3

Write a query against the rental table that returns the IDs of the
customers who rented a film on July 5, 2005 (use the rental.rental_date
column, and you can use the date() function to ignore the time
component). Include a single row for each distinct customer ID.

``` r
dbGetQuery(con,"
      PRAGMA table_info(rental)
           "
)
```

    ##   cid         name    type notnull dflt_value pk
    ## 1   0    rental_id INTEGER       0         NA  0
    ## 2   1  rental_date    TEXT       0         NA  0
    ## 3   2 inventory_id INTEGER       0         NA  0
    ## 4   3  customer_id INTEGER       0         NA  0
    ## 5   4  return_date    TEXT       0         NA  0
    ## 6   5     staff_id INTEGER       0         NA  0
    ## 7   6  last_update    TEXT       0         NA  0

``` r
dbGetQuery(con,"
SELECT DISTINCT customer_id, rental_date
FROM  rental
WHERE date(rental_date) = '2005-07-05'
")
```

    ##    customer_id         rental_date
    ## 1          565 2005-07-05 22:49:24
    ## 2          242 2005-07-05 22:51:44
    ## 3           37 2005-07-05 22:56:33
    ## 4           60 2005-07-05 22:57:34
    ## 5          594 2005-07-05 22:59:53
    ## 6            8 2005-07-05 23:01:21
    ## 7          490 2005-07-05 23:02:37
    ## 8          476 2005-07-05 23:05:17
    ## 9          322 2005-07-05 23:05:44
    ## 10         298 2005-07-05 23:08:53
    ## 11         382 2005-07-05 23:11:43
    ## 12         138 2005-07-05 23:13:07
    ## 13         520 2005-07-05 23:13:22
    ## 14         536 2005-07-05 23:13:51
    ## 15         114 2005-07-05 23:23:11
    ## 16         111 2005-07-05 23:25:54
    ## 17         296 2005-07-05 23:29:55
    ## 18         586 2005-07-05 23:30:36
    ## 19         349 2005-07-05 23:32:49
    ## 20         397 2005-07-05 23:33:40
    ## 21         369 2005-07-05 23:37:13
    ## 22         421 2005-07-05 23:41:08
    ## 23         142 2005-07-05 23:44:37
    ## 24         169 2005-07-05 23:46:19
    ## 25         348 2005-07-05 23:47:30
    ## 26         553 2005-07-05 23:50:04
    ## 27         295 2005-07-05 23:59:15

``` r
#dbGetQuery(con,"
#SELECT customer_id, 
#     COUNT(*) AS N
#FROM  rental
#WHERE date(rental_date) = '2005-07-05'
#GROUP BY customer_id
#")
```

# Exercise 4

Construct a query that retrieves all rows from the payment table where
the amount is either 1.99, 7.99, 9.99.

``` r
dbGetQuery(con,"
      PRAGMA table_info(payment)
")
```

    ##   cid         name    type notnull dflt_value pk
    ## 1   0   payment_id INTEGER       0         NA  0
    ## 2   1  customer_id INTEGER       0         NA  0
    ## 3   2     staff_id INTEGER       0         NA  0
    ## 4   3    rental_id INTEGER       0         NA  0
    ## 5   4       amount    REAL       0         NA  0
    ## 6   5 payment_date    TEXT       0         NA  0

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount IN (1.99, 7.99, 9.99)
LIMIT 10
  ")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16050         269        2         7   1.99 2007-01-24 21:40:19.996577
    ## 2       16056         270        1       193   1.99 2007-01-26 05:10:14.996577
    ## 3       16081         282        2        48   1.99 2007-01-25 04:49:12.996577
    ## 4       16103         294        1       595   1.99 2007-01-28 12:28:20.996577
    ## 5       16133         307        1       614   1.99 2007-01-28 14:01:54.996577
    ## 6       16158         316        1      1065   1.99 2007-01-31 07:23:22.996577
    ## 7       16160         318        1       224   9.99 2007-01-26 08:46:53.996577
    ## 8       16161         319        1        15   9.99 2007-01-24 23:07:48.996577
    ## 9       16180         330        2       967   7.99 2007-01-30 17:40:32.996577
    ## 10      16206         351        1      1137   1.99 2007-01-31 17:48:40.996577

# Ex 4.2

Construct a query that retrieves all rows from the payment table where
the amount is greater than 5

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount > 5
LIMIT 10
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16058         271        1      1096   8.99 2007-01-31 11:59:15.996577
    ## 3       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 4       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 5       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 6       16073         276        1       860  10.99 2007-01-30 01:13:42.996577
    ## 7       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 8       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 9       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 10      16087         286        2        81   6.99 2007-01-25 10:43:45.996577

Greater than 5 and less than 8

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount > 5   AND amount < 8
LIMIT 10
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 3       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 4       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 5       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 6       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 7       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 8       16087         286        2        81   6.99 2007-01-25 10:43:45.996577
    ## 9       16092         288        2       427   6.99 2007-01-27 14:38:30.996577
    ## 10      16094         288        2       565   5.99 2007-01-28 07:54:57.996577

# Exercise 5

Retrive all the payment IDs and their amount from the customers whose
last name is ‘DAVIS’.

``` r
dbGetQuery(con,"
      PRAGMA table_info(customer)
")
```

    ##    cid        name    type notnull dflt_value pk
    ## 1    0 customer_id INTEGER       0         NA  0
    ## 2    1    store_id INTEGER       0         NA  0
    ## 3    2  first_name    TEXT       0         NA  0
    ## 4    3   last_name    TEXT       0         NA  0
    ## 5    4       email    TEXT       0         NA  0
    ## 6    5  address_id INTEGER       0         NA  0
    ## 7    6  activebool    TEXT       0         NA  0
    ## 8    7 create_date    TEXT       0         NA  0
    ## 9    8 last_update    TEXT       0         NA  0
    ## 10   9      active INTEGER       0         NA  0

``` r
dbGetQuery(con,"
SELECT c.customer_id, c.last_name, p.payment_id, p.amount
FROM customer AS c INNER JOIN payment AS p
  ON c.customer_id = p.customer_id
 WHERE c.last_name == 'DAVIS' 
/* WHERE c.last_name == 'DAVIS' */  /* This is a comment*/
  ")
```

    ##   customer_id last_name payment_id amount
    ## 1           6     DAVIS      16685   4.99
    ## 2           6     DAVIS      16686   2.99
    ## 3           6     DAVIS      16687   0.99

# Exercise 6

## 6.1

Count the number of records in rental

``` r
dbGetQuery(con,"
SELECT  COUNT(*) AS count
FROM  rental
")
```

    ##   count
    ## 1 16044

## 6.2

Use COUNT(\*) and GROUP BY to count the number of rentals for each
customer_id

``` r
dbGetQuery(con,"
SELECT  customer_id, COUNT(*) AS count
FROM  rental
GROUP BY customer_id
LIMIT 8
")
```

    ##   customer_id count
    ## 1           1    32
    ## 2           2    27
    ## 3           3    26
    ## 4           4    22
    ## 5           5    38
    ## 6           6    28
    ## 7           7    33
    ## 8           8    24

## 6.3

Sort in descending order

``` r
dbGetQuery(con,"
SELECT  customer_id, COUNT(*) AS count
FROM  rental
GROUP BY customer_id
ORDER BY count DESC
LIMIT 8
")
```

    ##   customer_id count
    ## 1         148    46
    ## 2         526    45
    ## 3         236    42
    ## 4         144    42
    ## 5          75    41
    ## 6         469    40
    ## 7         197    40
    ## 8         468    39

## 6.4

Repeat the previous query but use HAVING to only keep the groups with 40
or more.

``` r
dbGetQuery(con,"
SELECT  customer_id, COUNT(*) AS count
FROM  rental
GROUP BY customer_id
HAVING count >= 40
ORDER BY count DESC
LIMIT 8
")
```

    ##   customer_id count
    ## 1         148    46
    ## 2         526    45
    ## 3         236    42
    ## 4         144    42
    ## 5          75    41
    ## 6         469    40
    ## 7         197    40

# Exercise 7

The following query calculates a number of summary statistics for the
payment table using MAX, MIN, AVG and SUM

``` r
dbGetQuery(con,"
SELECT MAX(amount) AS maxpayment,
       MIN(amount) AS minpayment,
       AVG(amount) AS avgpayment,
       SUM(amount) AS sumpayment
FROM payment
")
```

    ##   maxpayment minpayment avgpayment sumpayment
    ## 1      11.99       0.99   4.169775    4824.43

## 7.1 Modify the above query to do those calculations for each customer_id

``` r
dbGetQuery(con,"
SELECT customer_id,
       MAX(amount) AS maxpayment,
       MIN(amount) AS minpayment,
       AVG(amount) AS avgpayment,
       SUM(amount) AS sumpayment
FROM payment
GROUP BY customer_id
LIMIT 5
")
```

    ##   customer_id maxpayment minpayment avgpayment sumpayment
    ## 1           1       2.99       0.99   1.990000       3.98
    ## 2           2       4.99       4.99   4.990000       4.99
    ## 3           3       2.99       1.99   2.490000       4.98
    ## 4           5       6.99       0.99   3.323333       9.97
    ## 5           6       4.99       0.99   2.990000       8.97

## 7.2 do those calculations for each customer_id and save those for customers with more than 5 records

``` r
dbGetQuery(con,"
SELECT customer_id,
       COUNT(*)    AS  N,
       MAX(amount) AS maxpayment,
       MIN(amount) AS minpayment,
       AVG(amount) AS avgpayment,
       SUM(amount) AS sumpayment
FROM payment
GROUP BY customer_id 
HAVING N>5
")  
```

    ##    customer_id N maxpayment minpayment avgpayment sumpayment
    ## 1           19 6       9.99       0.99   4.490000      26.94
    ## 2           53 6       9.99       0.99   4.490000      26.94
    ## 3          109 7       7.99       0.99   3.990000      27.93
    ## 4          161 6       5.99       0.99   2.990000      17.94
    ## 5          197 8       3.99       0.99   2.615000      20.92
    ## 6          207 6       6.99       0.99   2.990000      17.94
    ## 7          239 6       7.99       2.99   5.656667      33.94
    ## 8          245 6       8.99       0.99   4.823333      28.94
    ## 9          251 6       4.99       1.99   3.323333      19.94
    ## 10         269 6       6.99       0.99   3.156667      18.94
    ## 11         274 6       5.99       2.99   4.156667      24.94
    ## 12         371 6       6.99       0.99   4.323333      25.94
    ## 13         506 7       8.99       0.99   4.132857      28.93
    ## 14         596 6       6.99       0.99   3.823333      22.94

``` r
# clean up
dbDisconnect(con)
```
