# Lecture 27 Databases 
## Example from mysql J documentation

[Connector/J Examples](https://dev.mysql.com/doc/connectors/en/connector-j-examples.html)
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.ResultSet;
// assume that conn is an already created JDBC connection (see previous examples)
Statement stmt = null;
ResultSet rs = null;
try {
    stmt = conn.createStatement();
    rs = stmt.executeQuery("SELECT foo FROM bar");
    // or alternatively, if you don't know ahead of time that
    // the query will be a SELECT...
    if (stmt.execute("SELECT foo FROM bar")) {
        rs = stmt.getResultSet();
    }
    // Now do something with the ResultSet ....
}
catch (SQLException ex){
    // handle any errors
    System.out.println("SQLException: " + ex.getMessage());
    System.out.println("SQLState: " + ex.getSQLState());
    System.out.println("VendorError: " + ex.getErrorCode());
}
finally {
    // it is a good idea to release
    // resources in a finally{} block
    // in reverse-order of their creation
    // if they are no-longer needed
    if (rs != null) {
        try {
            rs.close();
        } catch (SQLException sqlEx) { } // ignore
        rs = null;
    }
    if (stmt != null) {
        try {
            stmt.close();
        } catch (SQLException sqlEx) { } // ignore
        stmt = null;
    }
}
```

## SQL example


```sql
-- Create Database
CREATE DATABASE IF NOT EXISTS telran40;
USE telran40;

-- Create Personal Table
CREATE TABLE IF NOT EXISTS personal (
    person_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE,
    address VARCHAR(100)
);

-- Create Customer Table
CREATE TABLE IF NOT EXISTS customer (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    person_id INT,
    email VARCHAR(100),
    phone_number VARCHAR(20),
    FOREIGN KEY (person_id) REFERENCES personal(person_id)
);

-- Create Order Table
CREATE TABLE IF NOT EXISTS order_table (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);

-- Create Product Table
CREATE TABLE IF NOT EXISTS product (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(50),
    price DECIMAL(8, 2)
);

-- Create Order_Item Table
CREATE TABLE IF NOT EXISTS order_item (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT,
    subtotal DECIMAL(10, 2),
    FOREIGN KEY (order_id) REFERENCES order_table(order_id),
    FOREIGN KEY (product_id) REFERENCES product(product_id)
);

-- Insert Sample Data
INSERT INTO personal (first_name, last_name, birth_date, address)
VALUES ('John', 'Doe', '1990-01-01', '123 Main St'),
       ('Jane', 'Smith', '1985-05-15', '456 Oak St');

INSERT INTO customer (person_id, email, phone_number)
VALUES (1, 'john.doe@example.com', '555-1234'),
       (2, 'jane.smith@example.com', '555-5678');

INSERT INTO product (product_name, price)
VALUES ('Laptop', 999.99),
       ('Smartphone', 499.99);

INSERT INTO order_table (customer_id, order_date, total_amount)
VALUES (1, '2024-01-14', 1499.98),
       (2, '2024-01-15', 999.99);

INSERT INTO order_item (order_id, product_id, quantity, subtotal)
VALUES (1, 1, 2, 1999.98),
       (2, 2, 1, 499.99);

-- Create Views
CREATE VIEW customer_order_view AS
SELECT
    c.customer_id,
    CONCAT(p.first_name, ' ', p.last_name) AS customer_name,
    o.order_id,
    o.order_date,
    o.total_amount
FROM
    customer c
JOIN order_table o ON c.customer_id = o.customer_id
JOIN personal p ON c.person_id = p.person_id;

-- Create Indexes
CREATE INDEX idx_customer_email ON customer(email);
CREATE INDEX idx_product_name ON product(product_name);
CREATE INDEX idx_order_date ON order_table(order_date);

-- Example Join Query
SELECT *
FROM customer_order_view
WHERE order_date >= '2024-01-14';

```

## ERD - Entity Relationship Diagram

![NW5ym.png](src%2FNW5ym.png)


#### executeUpdate method:

_Purpose: Used for executing SQL statements that modify the database, such as INSERT, UPDATE, or DELETE statements. It is also used for executing SQL statements that do not return a result set.
Return Type: Returns an integer value representing the number of rows affected by the SQL statement. For INSERT, UPDATE, and DELETE statements, this value indicates the number of rows inserted, updated, or deleted._

##### executeQuery method:

_Purpose: Used for executing SQL queries that retrieve data from the database, typically SELECT statements.
Return Type: Returns a ResultSet object, which represents the result set of the query. You can use the methods of ResultSet to retrieve and process the data._


## Implementation in java 
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DatabaseExample {

    // JDBC URL, username, and password of MySQL server
    private static final String JDBC_URL = "jdbc:mysql://your-mysql-host:3306/telran40";
    private static final String USER = "telran40";
    private static final String PASSWORD = "telran40";

    public static void main(String[] args) {
        try {
            // Load the JDBC driver
            Class.forName("com.mysql.cj.jdbc.Driver");

            // Establish a connection
            try (Connection connection = DriverManager.getConnection(JDBC_URL, USER, PASSWORD)) {
                createTables(connection);
                insertSampleData(connection);
                executeQueries(connection);
            }
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
    }

    private static void createTables(Connection connection) throws SQLException {
        // Create tables
        try (Statement statement = connection.createStatement()) {
            statement.executeUpdate("CREATE TABLE IF NOT EXISTS personal (" +
                    "person_id INT PRIMARY KEY AUTO_INCREMENT," +
                    "first_name VARCHAR(50)," +
                    "last_name VARCHAR(50)," +
                    "birth_date DATE," +
                    "address VARCHAR(100))");

            statement.executeUpdate("CREATE TABLE IF NOT EXISTS customer (" +
                    "customer_id INT PRIMARY KEY AUTO_INCREMENT," +
                    "person_id INT," +
                    "email VARCHAR(100)," +
                    "phone_number VARCHAR(20)," +
                    "FOREIGN KEY (person_id) REFERENCES personal(person_id))");

            // Similar statements for other tables
        }
    }

    private static void insertSampleData(Connection connection) throws SQLException {
        // Insert sample data
        try (Statement statement = connection.createStatement()) {
            statement.executeUpdate("INSERT INTO personal (first_name, last_name, birth_date, address) " +
                    "VALUES ('John', 'Doe', '1990-01-01', '123 Main St'), " +
                    "('Jane', 'Smith', '1985-05-15', '456 Oak St')");

            statement.executeUpdate("INSERT INTO customer (person_id, email, phone_number) " +
                    "VALUES (1, 'john.doe@example.com', '555-1234'), " +
                    "(2, 'jane.smith@example.com', '555-5678')");

            // Similar statements for other tables
        }
    }

    private static void executeQueries(Connection connection) throws SQLException {
        // Execute queries
        try (Statement statement = connection.createStatement()) {
            // Example query using JOIN
            ResultSet resultSet = statement.executeQuery("SELECT c.customer_id, CONCAT(p.first_name, ' ', p.last_name) AS customer_name, " +
                    "o.order_id, o.order_date, o.total_amount " +
                    "FROM customer c " +
                    "JOIN order_table o ON c.customer_id = o.customer_id " +
                    "JOIN personal p ON c.person_id = p.person_id");

            // Process the result set
            while (resultSet.next()) {
                // Retrieve data from the result set
                int customerId = resultSet.getInt("customer_id");
                String customerName = resultSet.getString("customer_name");
                int orderId = resultSet.getInt("order_id");
                String orderDate = resultSet.getString("order_date");
                double totalAmount = resultSet.getDouble("total_amount");

                // Print or process the retrieved data as needed
                System.out.println("Customer ID: " + customerId + ", Customer Name: " + customerName +
                        ", Order ID: " + orderId + ", Order Date: " + orderDate + ", Total Amount: " + totalAmount);
            }
        }
    }
}

```

# Lecture 26 Databases 

## SQL Interview Questions

![SQL-Query-Interview-Question.jpg](src%2FSQL-Query-Interview-Question.jpg)

* Q. How to create new DB
```sql
CREATE DATABASE my_db;
```
```sql
CREATE DATABASE IF NOT EXISTS my_db;
```
<hr/>

* Q. How to check for existing databases
```sql
SHOW DATABASES;
```
<hr/>

* Q. How to switch to another db
```sql
USE my_another_db;
```
<hr/>


* Q. How to create new table in db
```sql
-- create a table Companies with name, id, address, email, and phone number
CREATE TABLE Companies (
  id int,
  name varchar(50),
  address text,
  email varchar(50),
  phone varchar(10)
);
```

```sql
-- create a Companies table if it does not exist
CREATE TABLE IF NOT EXISTS Companies (
  id int,
  name varchar(50),
  address text,
  email varchar(50),
  phone varchar(10)
);
```
```sql
-- create Colleges table with primary key college_id
CREATE TABLE Colleges (
  college_id INT,
  college_code VARCHAR(20) NOT NULL,
  college_name VARCHAR(50),
  CONSTRAINT CollegePK PRIMARY KEY (college_id)
);
```
<hr/>

* Q. How to DROP database
```sql
DROP DATABASE my_db;
```
<hr/>

* Q. How to DROP table
```sql
DROP TABLE my_table;
```
```sql
-- delete Orders table if it exists
DROP TABLE IF EXISTS Orders;
```
<hr/>


* Q. How to ALTER table

In SQL, the ALTER TABLE command is used to modify the structure of an existing table like adding, deleting, renaming columns, etc.
```sql
-- add phone column to Customers table
ALTER TABLE Customers
ADD phone varchar(10);
```
* - ALTER TABLE Operations

We can perform the following operations on a table using the ALTER TABLE command:

 - Add a column
 - Rename a column
 - Modify a column
 - Delete a column
 - Rename a table

```sql
-- add phone and age columns to Customers table
ALTER TABLE Customers
ADD phone varchar(10), age int;
```

```sql
-- rename column customer_id to c_id
ALTER TABLE Customers
RENAME COLUMN customer_id TO c_id;
```

```sql
ALTER TABLE Customers
MODIFY COLUMN age VARCHAR(2);
```

```sql
-- delete country column from Customers table
ALTER TABLE Customers
DROP COLUMN country;
```

```sql
-- rename Customers table to New_customers
ALTER TABLE Customers
RENAME TO New_customers;
```

* Q. What is the difference between SQL and MySQL?
* - SQL is a standard language which stands for Structured Query Language based on the English language
* - SQL is the core of the relational database which is used for accessing and managing database

* - MySQL is a database management system.
* - MySQL is an RDMS (Relational Database Management System) such as SQL Server, Informix etc.
<hr/>

* Q. What are the different subsets of SQL?
* - Data Definition Language (DDL) – It allows you to perform various operations on the database such as CREATE, ALTER, and DELETE objects.
* - Data Manipulation Language(DML) – It allows you to access and manipulate data. It helps you to insert, update, delete and retrieve data from the database.
* - Data Control Language(DCL) – It allows you to control access to the database. Example – Grant, Revoke access permissions.
![7uUaJ.png](src%2F7uUaJ.png)


<hr/>

* Q. What is the SELECT statement?
* - A SELECT command gets zero or more rows from one or more database tables or views. The most frequent data manipulation language (DML) command is SELECT in most applications. SELECT queries define a result set, but not how to calculate it, because SQL is a declarative programming language.
<hr/>

* Q. What are some common clauses used with SELECT query in SQL?
- The following are some frequent SQL clauses used in conjunction with a SELECT query:

* - WHERE clause: In SQL, the WHERE clause is used to filter records that are required depending on certain criteria.
```sql
-- select all columns from the customers table with last_name 'Doe' 
SELECT *
FROM Customers
WHERE last_name = 'Doe';
```
![sql-where-example.png](src%2Fsql-where-example.png)
- 
- ORDER BY clause: The ORDER BY clause in SQL is used to sort data in ascending (ASC) or descending (DESC) order depending on specified field(s) (DESC).
```sql
-- orders all rows from Customers in ascending order by country
SELECT *
FROM Customers
ORDER BY country;
```

```sql
-- orders all rows from Customers in ascending order by age 
SELECT *
FROM Customers
ORDER BY age ASC;
```

```sql
-- sort all rows from Customers, first by first_name and then by age
SELECT *
FROM Customers
ORDER BY first_name, age;
```

```sql
-- select last_name and age of customers who don't live in the UK
-- and sort them by last_name in descending order

SELECT last_name, age
FROM Customers
WHERE NOT country = 'UK'
ORDER BY last_name DESC;
```
* - GROUP BY clause: GROUP BY clause in SQL is used to group entries with identical data and may be used with aggregation methods to obtain summarised database results.
```sql
-- select the item column and the count of order ids from the Orders table
-- group them by the item column

SELECT COUNT(order_id), item
FROM Orders
GROUP BY item;
```

```sql
-- count the number of each country and group the rows by country
SELECT country, COUNT(*) AS number
FROM Customers
GROUP BY country;
```

```sql
-- select country, state, and minimum age from Persons table
-- group by country and state

SELECT country, state, MIN(age) AS min_age
FROM Persons
GROUP BY country, state;
```
![sql-group-by-example.png](src%2Fsql-group-by-example.png)

* - HAVING clause in SQL is used to filter records in combination with the GROUP BY clause. It is different from WHERE, since the WHERE clause cannot filter aggregated records.
```sql
-- select customers with the same first name based on their age count 
SELECT COUNT(age) AS Count, first_name
FROM Customers
GROUP BY first_name
HAVING COUNT(age) > 1;
```

```sql
-- select the count of customer ids greater than one and their corresponding country 
SELECT COUNT(customer_id), country
FROM Customers
GROUP BY country
HAVING COUNT(customer_id) > 1;
```
![sql-having.png](src%2Fsql-having.png)
<hr/>

* Q. What is PostgreSQL?
* - In 1986, a team lead by Computer Science Professor Michael Stonebraker created PostgreSQL under the name Postgres. It was created to aid developers in the development of enterprise-level applications by ensuring data integrity and fault tolerance in systems. PostgreSQL is an enterprise-level, versatile, resilient, open-source, object-relational database management system that supports variable workloads and concurrent users. The international developer community has constantly backed it. PostgreSQL has achieved significant appeal among developers because to its fault-tolerant characteristics.
It’s a very reliable database management system, with more than two decades of community work to thank for its high levels of resiliency, integrity, and accuracy. Many online, mobile, geospatial, and analytics applications utilise PostgreSQL as their primary data storage or data warehouse.
<hr/>

* Q. What are SQL comments?
* - SQL Comments are used to clarify portions of SQL statements and to prevent SQL statements from being executed. Comments are quite important in many programming languages. The comments are not supported by a Microsoft Access database. As a result, the Microsoft Access database is used in the examples in Mozilla Firefox and Microsoft Edge.
* - Single Line Comments: It starts with two consecutive hyphens (–).
* - Multi-line Comments: It starts with /* and ends with */.
<hr/>

* Q. What are Tables and Fields?

* - A table is a collection of data components organized in rows and columns in a relational database. A table can also be thought of as a useful representation of relationships. 
The most basic form of data storage is the table. An example is shown below.

![anatomy-of-a-sql-table-F-624x214.png](src%2Fanatomy-of-a-sql-table-F-624x214.png)
* - A Record or Row is a single entry in a table. In a table, a record represents a collection of connected data. The Employee table, for example, has four records.
* - A table is made up of numerous records (rows), each of which can be split down into smaller units called Fields(columns). ID, Name, Department, and Salary are the four fields in the Employee table above.
<hr/>

* Q. What is Database Black Box Testing?
* - Black Box Testing is a software testing approach that involves testing the functions of software applications without knowing the internal code structure, implementation details, or internal routes. Black Box Testing is a type of software testing that focuses on the input and output of software applications and is totally driven by software requirements and specifications. Behavioral testing is another name for it.
<hr/>

Q. What is an Index?
* - An index refers to a performance tuning method of allowing faster retrieval of records from the table. An index creates an entry for each value and hence it will be faster to retrieve data.
<hr/>

* Q. What is Normalization and what are the advantages of it?

Normalization in SQL is the process of organizing data to avoid duplication and redundancy. Some of the advantages are:
* - Better Database organization
* - More Tables with smaller rows
* - Efficient data access
* - Greater Flexibility for Queries
* - Quickly find the information
* - Easier to implement Security
* - Allows easy modification
* - Reduction of redundant and duplicate data
* - More Compact Database
* - Ensure Consistent data after modification
<hr/>

* Q. What is the difference between DROP and TRUNCATE commands?

DROP command removes a table and it cannot be rolled back from the database whereas TRUNCATE command removes all the rows from the table.
<hr/>


* Q. How many Aggregate functions are available in SQL?

SQL aggregate functions provide information about a database’s data. AVG, for example, returns the average of a database column’s values.

SQL provides seven (7) aggregate functions, which are given below:

* - AVG(): returns the average value from specified columns.
* - COUNT(): returns the number of table rows, including rows with null values.
```sql
-- returns the number of values in the Orders table
SELECT COUNT(*)
FROM Orders;
```
![sql-count.png](src%2Fsql-count.png)

```sql
-- count of customers who live in the UK
SELECT COUNT(country) AS customers_in_UK
FROM Customers
WHERE country = 'UK';
```
![sql-count-where.png](src%2Fsql-count-where.png)
* - MAX(): returns the largest value among the group.
```sql
SELECT MAX(column)
FROM table;
```
![sql-max.png](src%2Fsql-max.png)
* - MIN(): returns the smallest value among the group.
```sql
SELECT MIN(column)
FROM table;
```
* - SUM(): returns the total summed values(non-null) of the specified column.
* - FIRST(): returns the first value of an expression.
* - LAST(): returns the last value of an expression.
<hr/>

* Q. What is the default ordering of data using the ORDER BY clause? How could it be changed?

The ORDER BY clause in MySQL can be used without the ASC or DESC modifiers. The sort order is preset to ASC or ascending order when this attribute is absent from the ORDER BY clause.
<hr/>

* Q. How do we use the DISTINCT statement? What is its use?

The SQL DISTINCT keyword is combined with the SELECT query to remove all duplicate records and return only unique records. There may be times when a table has several duplicate records.
The DISTINCT clause in SQL is used to eliminate duplicates from a SELECT statement’s result set.
```sql
-- select the unique ages from the Customers table
SELECT DISTINCT age
FROM Customers;
```
![sql-select-distinct.png](src%2Fsql-select-distinct.png)

```sql
-- select rows if the first name and country of a customer is unique
SELECT DISTINCT country, first_name
FROM Customers;
```
![sql-select-distinct-2.png](src%2Fsql-select-distinct-2.png)



<hr>
<hr>
<hr>
SQL  — декларативный язык программирования, применяемый для создания, модификации и управления данными в реляционной базе данных, управляемой соответствующей системой управления базами данных.

SQL является, прежде всего, информационно-логическим языком, предназначенным для описания, изменения и извлечения данных, хранимых в реляционных базах данных. 

Шаг 1: Изучите основы SQL

Начните свое путешествие по SQL с освоения фундаментальных концепций. 

Начните с понимания базового синтаксиса, структуры базы данных и основных команд SQL, таких как SELECT, INSERT, UPDATE, DELETE и JOIN.

В телеграм t.me/sqlhub канале можно найти гайды, уроки, лучшие библиотеки и советы по работе с данными. 

https://t.me/addlist/_FjtIq8qMhU0NTYy -каналы по изучению Data Science и работе с базами данных.
 
1. Бесплатные курсы по SQL  - Интерактивный тренажер по SQL 

В курсе большинство шагов — это практические задания на создание SQL-запросов. Каждый шаг включает  минимальные теоретические аспекты по базам данных или языку SQL, примеры похожих запросов и пояснение к реализации.

Продолжительность: Приблизительно 2 недели

Ссылка - https://stepik.org/course/63054/promo

2. Введение в SQL от Kaggle

Базовый курс от известной платформы по анализу данных Kaggle.

Продолжительность: Приблизительно 1 неделя

Уровень мастерства: Начинающий

Ссылка - https://kaggle.com/learn/intro-to-sql

3. Продвинутый курс SQL от Kaggle

Продолжительность: Приблизительно 1 неделя

Уровень мастерства: Промежуточный

Ссылка - https://kaggle.com/learn/advanced-sql

4. Введение в базы данных и SQL-запросы от Udemy

Продолжительность: Приблизительно 1 неделя

Уровень мастерства: Начинающий

Ссылка - https://udemy.com/course/introduction-to-databases-and-sql-querying/

5. Intro to Relational Databases byUdacity

Продолжительность : 4 недели

Уровень квалификации: Средний ученик

Ссылка - https://udacity.com/course/intro-to-relational-databases-ud197

6. Введение в SQL (DataCamp)

Продолжительность: 4,5 часа

Уровень квалификации: Начинающий

Ссылка -

https://datacamp.com/courses/introduction-to-sql

7. SQL для анализа данных от Udacity

Продолжительность: Примерно 4 недели

Уровень навыков: Начинающий

Ссылка - https://shiksha.com/online-courses/sql-for-data-analysis-course-udacl3

8 - ЕЩЕ БЕСПЛАТНЫЕ курсы и БЕСПЛАТНЫЕ сертификаты.

❯ SQL http://cognitiveclass.ai/courses/learn-sql-relational-databases

❯ MySQL https://scaler.com/topics/course/sql-using-mysql-course/

❯ PostgreSQL http://freecodecamp.org/learn/relational-database/

Подборка ресурсов, где можно выучить\подтянуть знания SQL:

В порядке изучения с "нуля":

⏩ https://mode.com/sql-tutorial/  много бесплатных уроков для начинающих, идущих по нарастающей 

⏩ https://www.sql-ex.ru/ лучший тренажер по SQL, решайте парочку задач в день и никакие задания на интервью не будут вам страшны

⏩ https://sqlzoo.net/ еще несколько тренажеров с задачками для прокачивания практических навыков

⏩ https://stepik.org/course/70710/promo для тех кто уже знает основы и базу и хочет развить знания еще больше.

Шаг 2: Изучите различные типы СУБД

Базы данных SQL бывают разных типов. Очень важно понимать их различия. Среди популярных SQL-сервера - MySQL, PostgreSQL и Oracle. У каждого из них есть свои сильные стороны и возможности использования.

Например, MySQL известен своей простотой, а PostgreSQL предлагает расширенные возможности. Изучение этих различий поможет вам выбрать правильную базу данных для решения задач Data Science.

На Хабре есть замечательная статья на тему выбора СУБД.


❯ Oracle http://mygreatlearning.com/academy/learn-for-free/courses/oracle-sql

Ниже приведены некоторые из лучших ресурсов для поиска вопросов и практических заданий по SQL:

@data_analysis_ml -телеграм канал для Аналитиков данных со множеством гайдов и примеров с кодом и задачами по работе с данными.

SQLZoo - это бесплатный онлайн ресурс, который предлагает интерактивные уроки и задания для изучения SQL. Уроки начинаются с простых запросов и постепенно усложняются по мере продвижения в обучении.

W3Schools SQL - известный онлайн-ресурс, предлагающий уроки и примеры для изучения SQL и других языков программирования. Здесь пользователи могут найти множество материалов, которые помогут им углубить свои знания и применить их на практике.

Codecademy SQL - интерактивный курс для изучения SQL с возможностью практического применения на практике.

SQLBolt - это ресурс, который помогает начинающим и опытным пользователям SQL с помощью бесплатных уроков и задач.

Udacity SQL - курс  известного онлайн-образовательного ресурса, позволит вам освоить основы языка SQL и показать, как применять его для анализа данных

Khan Academy SQL - бесплатный курс SQL, предоставляющий уроки и задачи для изучения языка.

LearnSQL  - платный ресурс для изучения SQL. Содержит большое количество уроков и практических заданий.

SQLCourse - представляет собой бесплатную платформу, где можно овладеть навыками SQL. Здесь предоставлены обучающие уроки, практические задания и тесты, позволяющие проверить свои знания.

SQL Tutorial - это русскоязычный бесплатный ресурс, предоставляющий возможность изучения SQL. Здесь можно найти уроки и задания, которые помогут вам применять полученные знания на практике.

Mode Analytics SQL Tutorial - бесплатный курс, который предлагает обучение базовым и продвинутым навыкам работы с языком SQL.

SQL Exercises - это бесплатный онлайн-ресурс, который предлагает задачи и упражнения для изучения и практики SQL. Ресурс содержит множество заданий, которые помогут вам развить практические навыки работы с SQL.

SQL Fiddle – это интернет-сервис, который предоставляет возможность создавать, тестировать и отлаживать SQL-запросы совершенно бесплатно.

Learn SQL the Hard Way  - книга для изучения SQL, содержащая уроки и задания для практической работы.

DataCamp SQL - курс SQL от DataCamp, который научит Вас основам языка SQL и его применению в анализе данных. Содержит уроки и практические задания на практике.

5. Научитесь защищать свои базы данных
SQL базы данных являются одним из самых важных компонентов информационной системы любой организации. Они содержат в себе ценные данные, такие как персональная информация клиентов, финансовые данные, бизнес-планы и другую конфиденциальную информацию. Поэтому, защита SQL баз данных является критическим аспектом безопасности информационной системы.

Существует несколько способов защиты SQL баз данных:

Аутентификация и авторизация: Это первый шаг в обеспечении безопасности SQL баз данных. Аутентификация позволяет убедиться в подлинности пользователей, а авторизация определяет, какие действия могут выполнять эти пользователи в базе данных. Использование сложных паролей, двухфакторной аутентификации и ограничение прав доступа помогут предотвратить несанкционированный доступ к данным.

Шифрование данных: Шифрование данных является эффективным способом защиты SQL баз данных. Это процесс преобразования читаемых данных в непонятный для посторонних вид. Шифрование может быть применено как на уровне базы данных, так и на уровне приложения. Это поможет предотвратить утечку информации в случае несанкционированного доступа к базе данных.

Регулярные резервные копии: Регулярное создание резервных копий SQL баз данных является важным аспектом их защиты. В случае сбоя системы, атаки злоумышленников или случайного удаления данных, наличие резервной копии позволит быстро восстановить базу данных и минимизировать потери информации.

Обновление и патчи: Регулярное обновление программного обеспечения SQL баз данных и установка последних патчей являются неотъемлемой частью их защиты. Производители постоянно выпускают обновления, которые исправляют уязвимости и улучшают безопасность. Необходимо следить за выходом этих обновлений и устанавливать их как можно скорее.

Мониторинг и аудит: Регулярный мониторинг SQL баз данных позволяет обнаружить любые подозрительные активности или аномалии, которые могут указывать на возможные атаки или нарушения безопасности. Также важно вести аудит баз данных, чтобы иметь возможность отследить, кто и когда получал доступ к данным и какие действия совершал.

Все эти меры помогут обеспечить безопасность SQL баз данных и предотвратить несанкционированный доступ к ценной информации. Важно помнить, что безопасность баз данных является непрерывным процессом, требующим постоянного внимания и обновления.

Отдельно стоит поговорить про SQL-инъекции, вы возможно слышали этот термин, даже, если не работали с SQLдо этого.

SQL-инъекции (SQL injections, SQLi) — самый хорошо изученный и простой для понимания тип атаки на веб-сайт или веб-приложение. Тем не менее, он странным образом остается весьма распространенным и в наши дни. Организация OWASP (Open Web Application Security Project) упоминает SQL-инъекции в своем документе OWASP Top 10 2017 как угрозу номер один для безопасности веб-приложений, и вряд ли положение сильно изменилось за четыре года.SQL-инъекции 

Ниже приведены полезные инструменты для защиты от SQL инъекций

1.SuIP.biz

Обнаружение уязвимости для SQL-инъекций в режиме онлайн с помощью sup.biz и поддержка баз данных MySQL, Oracle, PostgreSQL, Microsoft SQL, IBM DB2, Firebird, Sybase.

SQLMap поможет протестировать сервис на все 6 методов инъекции.

2.Тест на уязвимость SQL-инъекции онлайнc HackerTarget 

Еще один онлайн-инструмент Hacker Target на основе SQLMap для поиска уязвимости bind & error против GET-запроса HTTP.

3. Netsparker

Netsparker готов просканировать уровень веб-безопасности предприятий: он делает даже больше, чем просто тест на уязвимость SQL. Человек также может интегрировать приложения для автоматизации веб-безопасности.

Пользователь может проверить индекс уязвимости сайта, который прошел сканирование от Netsparker.

4. Vega

Vega – это сканер безопасности с открытым исходным кодом, который может быть установлен на Linux, OS X и Windows.

Vega написан на Java, он имеет графический интерфейс.

Не только SQLi: Vega можно использовать для тестирования на многие другие типы уязвимостей, такие как:

Инъекция XML/Shell/URL;

Directory listing;

Remote file includes;

XSS.

Vega выглядит многообещающим бесплатным сканером безопасности сети.

5. SQLMap

SQLMap – это один из популярных инструментов тестирования с открытым исходным кодом на выполнение SQL-инъекций в системе управления реляционными базами данных.

Sqlmap проводит перечисление пользователей, паролей, хэшей, баз данных и поддерживает полный дамп таблиц базы данных.

Если пользователь использует Kali Linux, то он может применить SQLMap, не устанавливая его дополнительно.

6.SQL Injection Scanner

Онлайн сканер для проведения пентестинга, который использует OWASP ZAP. Есть две версии – упрощенная (бесплатная) и полная (нужно зарегистрироваться).

7.Appspider

Appspider, разработанный Rapid7, — это динамическое решение для тестирования безопасности приложений на обход защиты и более чем 95 типов атак.

Уникальная функция Appspider под названием «vulnerability validator» позволяет разработчику воспроизвести уязвимость в режиме реального времени.

Это очень удобно, когда администратор исправил уязвимость и хочет повторно протестировать ресурс, чтобы точно убедиться, что риска для системы больше нет.

8. Acunetix

Acunetix – это готовый к работе сканер уязвимостей веб-приложений, которому доверяют более 4000 компаний по всему миру. Не только сканирование SQLi: инструмент способен найти более 6000 других уязвимостей.

Каждая находка классифицируется, и показываются потенциальные корректировки системы безопасности: поэтому пользователь всегда знает, что нужно сделать, чтобы исправить ситуацию к лучшему. Кроме того, человек может интегрироваться с системой CI/CD и SDLC, поэтому каждый риск безопасности идентифицируется и фиксируется до того, как приложение будет развернуто.

9. Wapiti

Wapiti – это сканер уязвимостей на основе Рython. Он поддерживает большое количество инструментов для обнаружения следующих атак:

Sql и XPath;

CRLS и XSS;

Shellshock;

File disclosure;

Server-side request forgery;

Command execution.

Он поддерживает конечную точку HTTP/HTTPS, несколько типов аутентификации, такие как Basic, Digest, NTLM и Kerberos. У пользователя есть возможность создавать отчеты о сканировании в формате HTML, XML, JSON и TXT.

10. Scant3r

Scant3r – это «легкий сканер», основанный на Python.

Он ищет возможность проведения атак XSS, SQLi, RCE, SSTI в заголовках и параметрах URL-адресов.


❯ PostgreSQL http://simplilearn.com/free-postgresql-course-skillup

❯ SQL Projects http://mygreatlearning.com/academy/learn-for-free/courses/sql-projects-for-beginners


