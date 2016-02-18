SimpleJDBC is a JDBC driver for [Amazon's SimpleDB](http://aws.amazon.com/simpledb/). SimpleJDBC makes it possible to use the myriad tools available that leverage standard JDBC interfaces to facilitate working with relational data (i.e. SimpleJDBC enables ETL tools, like [Scriptella](http://scriptella.javaforge.com/) to import or extract data from SimpleDB).

Keep in mind, SimpleDB isn't a traditional relational database -- in fact, [it's quite different](http://www.ibm.com/developerworks/java/library/j-javadev2-9/). Nevertheless, it is possible to map basic SQL commands (and therefore JDBC) to the core API offered by Amazon. Using SimpleJDBC is straightforward. All that is required is Java 5 and the jar files provided in the download.

You load SimpleJDBC like you would any other JDBC driver.

```
Properties prop = new Properties();
prop.setProperty("secretKey", ...);
prop.setProperty("accessKey", ...);

Class.forName("com.beacon50.jdbc.aws.SimpleDBDriver");
Connection con = DriverManager.getConnection("jdbc:simpledb://sdb.amazonaws.com", prop);
```

Note, with Amazon's web services, there is no notion of a username or password -- authorization is provided by keys; thus, to use SimpleJDBC, you must provide your [Amazon secret key and your access key](http://docs.amazonwebservices.com/AWSSecurityCredentials/1.0/AboutAWSCredentials.html).

With a `Connection` instance, you can issue queries like `SELECT`, `UPDATE`, `DELETE`, and `INSERT` -- you can use JDBC `Statement`s or even `PreparedStatement`s (although, the notion of a pre-compiled SQL query in SimpleDB doesn't exist; consequently, there isn't any affect on performance).

For example, SQL `INSERT`s are handled just like normal:

```
Statement st = conn.createStatement();
String insert = "INSERT INTO users (name, age) VALUES ('Ann Smith', 33)";
int val = st.executeUpdate(insert);
```

or

```
PreparedStatement pstmt = conn.prepareStatement("INSERT INTO users (name, age) VALUES (?, ?)");
pstmt.setString(1, "Annie Smith");
pstmt.setInt(2, 33);
int val = pstmt.executeUpdate();
```

Note, because SimpleDB lacks rich data types (everything is a string), SimpleJDBC attempts to encode and decode numeric values. Thus, in the two `INSERT` statements above, 33 will be zero padded when stored in SimpleDB (i.e. 000033) and decoded back to 33 upon a `SELECT` query.

You can use the all too familiar JDBC `ResultSet` too:

```
String qry = "select * from users where name = 'Joe Smith'";
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery(qry);

while (rs.next()) {
    int iage = rs.getInt("age");
    //...
}
```

Most basic SQL/JDBC features have been implemented -- you can [view various test cases](https://github.com/beacon50/SimpleJDBC) for more details. Note -- using SimpleDB forces you to understand [eventual consistency](http://www.allthingsdistributed.com/2007/12/eventually_consistent.html); that is, ACID-ness isn't supported in SimpleDB. Nevertheless, in exchange for consistency (the "C" in ACID) you get massive scalability + reliability. In practice, this means you might run into cases where an immediate `SELECT` following an `INSERT` might not return a result.  Consequently, you'll see that all test cases in SimpleJDBC sleep so as to provide for a deterministic situation in a JUnit environment.