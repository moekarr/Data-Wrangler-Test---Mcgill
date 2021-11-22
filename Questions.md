# Data-Wrangler-Test---Mcgill
# Question 1
### Implement a database model for this dataset. Share your database design as an entity-relationship diagram, and as a script to automate its creation in an SQL database.
![Capture](https://user-images.githubusercontent.com/93660460/142931607-dcf266c3-3d99-4093-9d3c-6cdbff2f0f0b.JPG)

## SQL CODE

```
-- we will create the 3 tables structure by defining each data type to be input later by python.

CREATE TABLE customer (firstname VARCHAR(100) NOT NULL,
                       lastname VARCHAR(100) NOT NULL,
                       email VARCHAR(250) PRIMARY KEY UNIQUE NOT NULL,	
                       streetnumber INTEGER,streetname VARCHAR(250),
                       city VARCHAR(250),phone VARCHAR(20) unique not null);


CREATE TABLE Products(product_number varchar(250) PRIMARY KEY,
                      department VARCHAR(100) NOT NULL,
	                    price money not null);



CREATE TABLE Purchases(purchase_date date not null,
                       product_number VARCHAR(250) references products(product_number),
	                      email VARCHAR(250) not null references customer(email));
```

# Question 2
## PYTHON CODE
### the database with the data provided in the CSV files using Python or Bash.
```
# import the necessary libraries, psycopg2 is for linking python and sql
import psycopg2 as pg2
import pandas as pd
# connection my database in postgres to python.
conn=pg2.connect(database='Data Wrangler',user='postgres',password='12345678')
cur = conn.cursor()
# reading the Customers csv file 
df_state=pd.read_csv('Customers.csv')
# finding if there is any duplicates
df_state[df_state.duplicated(['email'],keep=False)]
# dropping out the duplicates, keeping the first ones,since the second duplicate email shouldnt be allowed.
df_state.drop_duplicates(keep='first',subset=['email'],inplace=True)
df_state.drop(df_state.columns[df_state.columns.str.contains('Unnamed',case = False)],axis = 1, inplace = True)
# saving the new dataset to the csv file
df_state.to_csv('Customers.csv',index=False)
df_state
# inserting the CSV file into the customer column in sql
with open('Customers.csv', 'r',encoding='utf-8') as g:
    next(g) #removing headers
    cur.copy_from(g, 'customer', sep=",")

conn.commit()

# inserting the products CSV file into the products column in sql
with open('products.csv', 'r',encoding='utf-8') as f:
    next(f) #removing headers
    cur.copy_from(f, 'products', sep=",")
conn.commit()

# reading the purchases csv file 
df_state2=pd.read_csv('purchases.csv')
df_state2
# removing the unnamed column (blank columns)
df_state2.drop(df_state2.columns[df_state2.columns.str.contains('Unnamed',case = False)],axis = 1, inplace = True)
# saving the new dataset to the csv file
df_state2.to_csv('purchases.csv',index=False)
# inserting the purchases CSV file into the purchases column in sql
with open('purchases.csv', 'r',encoding='utf-8') as h:
    next(h)
    cur.copy_from(h, 'purchases', sep=",")
conn.commit()


```
# Question 3
## SQL CODE
### What would be the SQL query to display all the orders (assuming an order is grouping all the purchases from the same customer on the same day)?
```
select firstname,lastname,customer.email,purchase_date,count(*) from customer
join purchases
on customer.email=purchases.email
group by customer.email,purchase_date
order by email

-- we can also use the code below sort out customer and their orders.
select firstname,lastname,customer.email,purchase_date,product_number from customer
join purchases
on customer.email=purchases.email
order by email, purchase_date;
```

# Question 4
### How would you optimize this database if it gets larger and the queries are becoming slow to execute?

As the database becomes larger and queries become slower to execute it becomes even more important to write more efficient queries to retrieve data. We should start by query optimization and also index management. So first we should ask ourselves what were the most expensive queries running when things were slow? So we need to investigate what was happening on the SQL Server over the time the application was running very slowly. Generally continuous monitoring is very important to delivering consistent performance across all SQL Server databases.
Here are Some possible solutions to increase performance: 

1-We have to know that generally our goal is to index the major searching and ordering columns.  Proper indexing ensures quicker access to the database. This means that once you’ve created an index, you can select or sort your rows faster than before. Indexes are also used to define a primary-key or unique index which will make sure that no other columns have the same values. So creating index of filtered columns when the query returns lots of data is a solution. Basically, the goal is to index the major searching and ordering columns.

Also usually we have to drop the SQL indexes before performing huge inserts of million-plus rows to speed up the insertion process. After the batch is inserted, we can redefine the indexes. We can always try to merge as many indexes as you we can and delete the indexes we don’t need.

2- We should avoid using large queries that use loops. That could decrease the SQL performance significantly. We could use INSERT statement with multiple rows instead.

3- We should use INNER JOINS rather than WHERE clauses, because in WHERE clause,it creates all possible combinations of the variables created. They would be problematic especially in large databases. 
Also instead of using subqueires which tend to run row-by-row once for each row returned by the outer query (usually using WHERE from outer query), A more efficient SQL performance technique would be to rewrite the correlated subquery as a JOIN.

4- One other thing to have in mind in SQL optimization is to avoid using SELECT * . if a table has too many rows, this slows down queries by querying a lot of unnecessary data. Instead we can individually include the specific columns that we need.  Another example is that we can use the LIMIT command when we need data from a certain number of rows from among the lot.  Using a LIMIT statement prevents the database to do a large query run, only to find out the query that needs editing. Also we have to have 
in mind that if we are using wildcards such as LIKE to used them wisely and be more specific as it will pull out unnecessary results after searching all records.

5- Another thing to have in mind is to select datatypes wisely if we are to design a new input to the database. For example you should use the column type to INT instead of VARCHAR, if the column just contains numbers. or for example using CHAR instead of VARCHAR for string values that are all two characters. CHAR is fixed length and VARCHAR is variable length. 

