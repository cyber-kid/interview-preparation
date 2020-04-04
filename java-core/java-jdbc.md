# JDBC
* [Introducing the Interfaces of JDBC](#introducing-the-interfaces-of-jdbc)
* [Building a JDBC URL](#building-a-jdbc-url)
* [Getting a Database Connection](#getting-a-database-connection)
* [Obtaining a Statement](#obtaining-a-statement)
* [Choosing a ResultSet Type](#choosing-a-resultset-type)
* [Choosing a ResultSet Concurrency Mode](#choosing-a-resultset-concurrency-mode)
* [Executing a Statement](#executing-a-statement)
* [PreparedStatement](#preparedstatement)
* [CallableStatement](#callablestatement)
* [Reading a ResultSet](#reading-a-resultset)
* [Getting Data for a Column](#getting-data-for-a-column)
* [Scrolling ResultSet](#scrolling-resultset)
* [Closing Database Resources](#closing-database-resources)
* [Dealing with Exceptions](#dealing-with-exceptions)
* [Updating and Inserting rows](#updating-and-inserting-rows)
#### Introducing the Interfaces of JDBC
JDBC consists of four key interfaces. The interfaces are declared in the JDK. As you know, interfaces need a concrete class to implement them in order to be useful. These concrete classes come from the JDBC driver. Each database has a different JAR file with these classes. With JDBC, you use only the interfaces in your code and never the implementation classes directly. In fact, they might not even be public classes.

Four JDBC interfaces:
* **Driver**: Knows how to get a connection to the database
* **Connection**: Knows how to communicate with the database
* **Statement**: Knows how to run the SQL
* **ResultSet**: Knows what was returned by a SELECT query

Code sample:
```java
package com.wiley.ocp.connection;
import java.sql.*;
public class MyFirstDatabaseConnection {
   public static void main(String[] args) throws SQLException {
      String url = "jdbc:derby:zoo";
      try (Connection conn = DriverManager.getConnection(url);
           Statement stmt = conn.createStatement();
           ResultSet rs = stmt.executeQuery("select name from animal")) {
         while (rs.next()) System.out.println(rs.getString(1));
      }
   }
}
```
#### Building a JDBC URL
All URL used to connect to db have three parts in common. The first piece is always the same. It is the protocol jdbc. The second part is the name of the database such as derby, mysql, or postgres. The third part is “the rest of it,” which is a database-specific format. Colons separate the three parts. The third part typically contains the location and the name of the database. The syntax varies depending on a vendor.

Example:
```java
jdbc:oracle:thin:@123.123.123.123:1521:zoo
```
#### Getting a Database Connection
There are two main ways to get a Connection: DriverManager or DataSource. Do not use a DriverManager in code someone is paying you to write. A DataSource is a factory, and it has more features than DriverManager. For example, it can pool connections or store the database connection info outside the application. The DriverManager class is in the JDK, as it is an API that comes with Java. The method has an easy-to-remember name—getConnection().

To get a Connection from the embedded database, you write the following:
```java
import java.sql.*;

public class TestConnect {
   public static void main(String[] args) throws SQLException {
      Connection conn = DriverManager.getConnection("jdbc:derby:zoo");
      System.out.println(conn);
   }
}
```
Running this example as java TestConnect will give you an error that begins with this:
```
Exception in thread "main" java.sql.SQLException: No suitable driver found for jdbc:derby:zoo
at java.sql.DriverManager.getConnection(DriverManager.java:689) at java.sql.DriverManager.getConnection(DriverManager.java:270)
```
The class SQLException means “something went wrong when connecting to or accessing the database.” In this case, we didn’t tell Java where to find the database driver JAR file. Remember that the implementation class for Connection is found inside a driver JAR. We try this again by adding the classpath with:
```java
java -cp "<java_home>/db/lib/ derby.jar:." TestConnect.
```
Remember to substitute the location of where Java is installed on your computer for <java_home>. (If you are on Windows, replace the colon with a semicolon.)

There is also a signature that takes a username and password:
```java
import java.sql.*;

public class TestExternal {
   public static void main(String[] args) throws SQLException {
      Connection conn = DriverManager.getConnection("jdbc:postgresql://localhost:5432/ocp-book", "username", "password");
      System.out.println(conn);
   }
}
```
#### Obtaining a Statement
In order to run SQL, you need to tell a Statement about it. Getting a Statement from a Connection is easy:
```java
Statement stmt = conn.createStatement();
```
There’s another one that takes two parameters. The first is the ResultSet type, and the other is the ResultSet concurrency mode.
```java
Statement stmt = conn.createStatement( ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
```
#### Choosing a ResultSet Type
By default, a ResultSet is in TYPE_FORWARD_ONLY mode. You can go through the data once in the order in which it was retrieved. Two other modes are TYPE_SCROLL_INSENSITIVE and TYPE_SCROLL_SENSITIVE. Both allow you to go through the data in any order. You can go both forward and backward. You can even go to a specific spot in the data. The difference between TYPE_SCROLL_INSENSITIVE and TYPE_SCROLL_SENSITIVE is what happens when data changes in the actual database while you are busy scrolling. With TYPE_ SCROLL_INSENSITIVE, you have a static view of what the ResultSet looked like when you did the query. If the data changed in the table, you will see it as it was when you did the query. With TYPE_SCROLL_SENSITIVE, you would see the latest data when scrolling through the ResultSet. This one is not widely supported. If the type you request isn’t available, the driver can “helpfully” downgrade to one that is. This means that if you ask for TYPE_SCROLL_SENSITIVE, you will likely get a Statement that is TYPE_SCROLL_INSENSITIVE.
#### Choosing a ResultSet Concurrency Mode
By default, a ResultSet is in CONCUR_READ_ONLY mode. It means that you can’t update the result set. The second one lets you modify the database through the ResultSet. It is called CONCUR_UPDATABLE. It is a bad coding practice to use second type.Again, if the mode you request isn’t available, the driver can downgrade you.
#### Executing a Statement
SQL statements that begin with DELETE, INSERT, or UPDATE typically use a method called executeUpdate(). The method takes the SQL statement to run as a parameter. It returns the number of rows that were inserted, deleted, or changed. Here’s an example of all three update types:
```java
Statement stmt = conn.createStatement();

int result = stmt.executeUpdate("insert into species values(10, 'Deer', 3)");
System.out.println(result); // 1

result = stmt.executeUpdate("update species set name = '' where name = 'None'");
System.out.println(result); // 0

result = stmt.executeUpdate("delete from species where id = 10");
System.out.println(result); // 1
```
To run SELECT queries excuteQuery() method should be used(the return type is ResultSet):
```java
ResultSet rs = stmt.executeQuery("select * from species");
```
There’s a third method called execute() that can run either a query or an update. It returns a boolean so that we know whether there is a ResultSet. That way, we can call the proper method to get more detail. The pattern looks like this:
```java
boolean isResultSet = stmt.execute(sql);
if (isResultSet) {
   ResultSet rs = stmt.getResultSet();
   System.out.println("ran a query");
} else {
   int result = stmt.getUpdateCount();
   System.out.println("ran an update");
}
```
If we use the wrong method for a SQL statement we get a runtime exception: SQLException.
#### PreparedStatement
In real life, you should not use Statement directly. You should use a subclass called PreparedStatement. This subclass has three advantages:performance, security, and readability.
* **Performance**: In most programs you run similar queries multiple times. A PreparedStatement figures out a plan to run the SQL well and remembers it.
* **Security**: Avoids SQL-Injections by taking care of all the escaping. Example:
```java
PreparedStatement ps = conn.prepareStatement("delete from animal where name = ?");
ps.setString(1, name);
ps.execute();
```
* **Readability**: It’s nice not to have to deal with string concatenation in building a query string with lots of variables.
#### CallableStatement
A CallableStatement is meant for executing a stored procedure that has already been created in the database. For example:
```java
//computeMatrixForSales is a stored procedure that has already been created in the database.
  callableStatement = connection.prepareCall("{call computeMatrixForSales(?)});
  callableStatement.setInt(1, 1000);
  callableStatement.executeUpdate();
```
#### Reading a ResultSet
When working with a forward-only ResultSet, most of the time you will write a loop to look at each row. The code looks like this:
```java
Map<Integer, String> idToNameMap = new HashMap<>();
ResultSet rs = stmt.executeQuery("select id, name from species");
while(rs.next()) {
   int id = rs.getInt("id");
   String name = rs.getString("name");
   idToNameMap.put(id, name);
}
System.out.println(idToNameMap); // {1=African Elephant, 2=Zebra}
```
There is another way to access the columns. You can use an index instead of a column name.
```java
Map<Integer, String> idToNameMap = new HashMap<>();
ResultSet rs = stmt.executeQuery("select id, name from species");
while(rs.next()) {
   int id = rs.getInt(1);
   String name = rs.getString(2);
   idToNameMap.put(id, name);
}
System.out.println(idToNameMap); // {1=African Elephant, 2=Zebra}
```
It is very important to check that rs.next() returns true before trying to call a getter on the ResultSet. Attempting to access a column that does not exist throws a SQLException, as does getting data from a ResultSet when it isn’t pointing at a valid row.

Not calling rs.next() at all is a problem. The result set cursor is still pointing to a location before the first row, so the getter has nothing to point to. Example:
```java
ResultSet rs = stmt.executeQuery("select count(*) from animal");
rs.getInt(1); // throws SQLException
```
To sum up this section, it is very important to remember the following:
* Always use an if statement or while loop when calling rs.next().
* Column indexes begin with 1.
#### Getting Data for a Column
To get Date values from DB:
```java
ResultSet rs = stmt.executeQuery("select date_born from animal where name = 'Elsa'");
if (rs.next()) {
   java.sql.Date sqlDate = rs.getDate(1);
   LocalDate localDate = sqlDate.toLocalDate();
   System.out.println(localDate); // 2001―05―06
}
```
To get TIme value from DB:
```java
ResultSet rs = stmt.executeQuery("select date_born from animal where name = 'Elsa'");
if (rs.next()) {
   java.sql.Time sqlTime = rs.getTime(1);
   LocalTime localTime = sqlTime.toLocalTime();
   System.out.println(localTime); // 02:15
}
```
To get Timestamp value from DB:
```java
ResultSet rs = stmt.executeQuery("select date_born from animal where name = 'Elsa'");
if (rs.next()) {
   java.sql.Timestamp sqlTimeStamp = rs.getTimestamp(1);
   LocalDateTime localDateTime = sqlTimeStamp.toLocalDateTime();
   System.out.println(localDateTime); // 2001―05―06T02:15
}
```
#### Scrolling ResultSet
There’s also a previous() method, which moves backward one row and returns true if pointing to a valid row of data. There are also methods to start at the beginning and end of the ResultSet. The first() and last() methods return a boolean for whether they were successful at finding a row. The beforeFirst() and afterLast() methods have a return type of void, since it is always possible to get to a spot that doesn’t have data. beforeFirst() and afterLast() don’t point to rows in the ResultSet. Another method is absolute() takes the row number to which you want to move the cursor as a parameter. A positive number moves the cursor to that numbered row. Zero moves the cursor to a location immediately before the first row. A negative number means to start counting from the end of the ResultSet rather than from the beginning. It returns a boolean for whether it was successful at finding a row. There is a relative() method that moves forward or backward the requested number of rows. It returns a boolean if the cursor is pointing to a row with data.
#### Closing Database Resources
Example:
```java
public static void main(String[] args) throws SQLException {
   String url = " jdbc:derby:zoo";
   try (Connection conn = DriverManager.getConnection(url);
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("select name from animal")) {
      while (rs.next()) System.out.println(rs.getString(1));
   }
}
```
Remember that a try-with-resources statement closes the resources in the reverse order from which they were opened. This means that the ResultSet is closed first, followed by the Statement, and then the Connection. This is the standard order to close resources.

Closing a JDBC resource should close any resources that it created. In particular, the following are true:
* Closing a Connection also closes the Statement and ResultSet.
* Closing a Statement also closes the ResultSet.

There’s another way to close a ResultSet. JDBC automatically closes a ResultSet when you run another SQL statement from the same Statement.
#### Dealing with Exceptions
```java
String url = " jdbc:derby:zoo";
try (Connection conn = DriverManager.getConnection(url ");
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("select not_a_column from animal")) {

  while (rs.next()) System.out.println(rs.getString(1));
} catch (SQLException e) {
   System.out.println(e.getMessage());
   System.out.println(e.getSQLState());
   System.out.println(e.getErrorCode());
}
```
The output is:
ERROR: column "not_a_column" does not exist Position: 8
42703
0

Used methods:
* The **getMessage()** method returns a human-readable message as to what went wrong.
* The **getSQLState()** method returns a code as to what went wrong. You can Google the name of your database and the SQL state to get more information about the error.
* The **getErrorCode()** is a database-specific code. On this database, it doesn’t do anything.
#### Updating and Inserting rows
JDBC 2.0 allows you to use ResultSet object to update an existing row and even insert new row in the database. For both the cases, the ResultSet must be updatable, which can be achieved by passing ResultSet.CONCUR_UPDATABLE while creating a Statement object:
```java
stmt = con.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);
```
or
```java
stmt = con.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE);
```
The general usage pattern for this functionality is as follows:

To update an existing row:
1. First, go to the row you want to update. You can either iterate through a ResultSet to reach a particular row or just call rs.absolute(int rowNumber).
2. Now update the columns of the ResultSet with required values using rs.updateXXX(columnNumber, value) or rs.updateXXX(columnName, value) methods.
3. Call rs.updateRow(); If you call rs.refreshRow() without calling updateRow(), the updates will be lost.

To insert a new Row:
1. Call rs.moveToInsertRow(); first. You can't insert a row without calling this method first.
2. Use rs.updateXXX methods to update all column values. You must set values for all the columns.
3. Call rs.insertRow(); 
4. Call rs.moveToCurrentRow(); to go back to the row where you were before calling moveToInsertRow.