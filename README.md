# DB-Assignment6
[Database Assignment 6](https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment6.md)

------
### Pre-setup
Remember to `stop` any previous containers that are running on port 3306 

`docker stop [container]`

Command to see containers:

```
docker ps
docker ls
docker container ls --all
```

------
## Setup

#### The Docker Container
Assignment done using Vagrant, Docker and Workbench

Run the following command with Docker to create the container:

`docker run --name my_db6mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=iphone2019 -d mysql`


Access the Docker container:

`docker exec -it my_db6mysql bash`


Within the Docker Container, run the following 2 commands to update the container and download 7zip:

```shell
apt-get update

apt-get install wget p7zip-full -y

apt-get install unzip
```


Download the Data **Heads-up: You might want to start making some coffee while the data is being downloaded**:

```
wget https://archive.org/download/stackexchange/coffee.stackexchange.com.7z

wget http://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip

wget https://raw.githubusercontent.com/radeonxray/DB-Assignment6/master/CreateTables.sql
```


Extract the Data **Heads-up: You might want to start running a marathon while the data is being extracted**:

```
7z e coffee.stackexchange.com.7z 
unzip mysqlsampledatabase.zip
```

-----

#### Connect to the Database through WorkBench

To connect to the Database through Workbench, make sure you have the latest version of the [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

My Default information to connect to the Docker Container:

*IP*: `192.168.33.10`

*Port*: `3306`

*User*: `root`

*Password*: `iphone2019`

-----

#### Setup Mysql And The Database 

Start MySQL in the container:

`mysql -u root -piphone2019 --local-infile`

When inside the mysql:

```
source ./CreateTables.sql;
source ./mysqlsampledatabase.sql

use classicmodels;
```




-----

## Excercise 1

In the `classicmodels` database, write a query that picks out those customers who are in the same city as office of their sales representative.

### Hand-in:
Insert into the readme file: the query and the graphical execution plan which can be obtained from the query. Explain what is the main performance problem for this query. Do not try to optimize the database for this query (yet).

```mysql
SELECT customers.* FROM customers
INNER JOIN employees ON employees.employeeNumber = customers.salesRepEmployeeNumber
INNER JOIN offices ON offices.officeCode = employees.officeCode
WHERE customers.city = offices.city
```

### Review:
* Are you able to use the query?
* Do you have the same explanation?
* If you find the explanation good or bad - say so, and be constructive.

-------------------------

## Exercise 2
Change the database schema so that the query from exercise get better performance. 

### Hand-in:
Explain in the readme file what changes you did, if you changed the query or the schema. Insert a new graphical execution plan, and point out in the readme file why this new one is better.

```mysql
Create index office_city ON offices (city);
Create index customer_city ON customers (city);
```

### Review:
* Are you able to reproduce the speedup?
* Do you agree with the explanation?
* If you find the explanation good or bad - say so, and be constructive.

## Exercise 3
We want to find out how much each office has sold and the max single payment for each office. Write two queries which give this information

a) using grouping<br>
b) using windowing

For each of the two solutions, check its graphical execution plan.

### Hand-in:
The two queries and the graphical execution plans. Explain any differences and try to explain why there is or is not any difference.

Using Grouping
```mysql
SELECT offices.officeCode, sum(payments.amount) as paymentPrice, max(payments.amount) as maxSingle FROM payments
INNER JOIN customers ON payments.customerNumber = customers.customerNumber
INNER JOIN employees ON customers.salesRepEmployeeNumber = employees.employeeNumber
INNER JOIN offices ON employees.officeCode = offices.officeCode
GROUP BY offices.officeCode
ORDER BY paymentPrice DESC;
```

Using Windowing
```mysql
select offices.city as "officeCity", e.firstName as "employ", c.customerName, p.paymentDate, p.amount,
sum(p.amount) over (partition by offices.officeCode) as "sumForOffice",
max(p.amount) over (partition by offices.officeCode) as "maxForOffice"
from offices
left join employees e on offices.officeCode = e.officeCode
left join customers c on e.employeeNumber = c.salesRepEmployeeNumber
left join payments p on c.customerNumber = p.customerNumber
where p.amount is not null;
```

### Review:
* Are you able to reproduce the difference?
* Do you agree with the explanation?
* If you find the explanation good or bad - say so, and be constructive.

-------------------------

## Exercise 4
In the stackexchange forum for coffee (coffee.stackexchange.com), write a query which return the displayName and title of all posts which with the word `grounds`in the title.

> If you want to challenge yourself a bit, use the ubuntu stackexchange instead, and search for `grep` rather than `grounds` in this and exercise 5.

### Hand-in:
* Hand in the query. Show the execution plan for the query (if you cannot get the graphical, show the tabular).
* Document that there is no real cost to the join to get the display name instead of just the userid. You can do that by running an other query with no join and then show that there is no major difference.

```mysql
SELECT DisplayName, Title FROM posts INNER JOIN users ON posts.OwnerUserId = users.Id where Title LIKE '%grounds%' 
```

### Review:
* Are you able to verify there is no major difference?
* Do you agree with the explanation?
* If you find the explanation good or bad - say so, and be constructive.

-------------------------

## Exercise 5
Add a full text index to the `posts` table and change the query from exercise 4 so it no longer scans the entire `posts` table. 

### Hand-in:
* the revised query
* the sql needed to add your index
	* in particular your choice between a "natural language" full-text search and a "boolean" full-text search.
* documentation of efficiency in the form of an execution plan

```mysql
ALTER TABLE posts  
ADD FULLTEXT(Title)
```

Then

```mysql
SELECT DisplayName, Title FROM posts INNER JOIN users ON posts.OwnerUserId = users.Id WHERE MATCH(Title) AGAINST ('grounds' IN natural language mode)

```

### Review:
* Are you able to reproduce the difference?
* Do you belive this is the optimal query?
* If you belive you have a better solution, say so - and be constructive.

-------------------------
