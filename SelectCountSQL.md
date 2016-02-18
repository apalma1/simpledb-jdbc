# Introduction #

SimpleJDBC supports the standard SQL `select count(*) from YOUR_DOMAIN`. The resultant `ResultSet` will contain a record named `count`, which you can then retrieve via a `getInt` call.

```
String qry = "select count(*) from users_tst";
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery(qry);

while (rs.next()) {
   int icnt = rs.getInt("count");
  //do something...
}
```