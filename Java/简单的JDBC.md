---
title: 简单的JDBC
categories: Java
tags: []
---

```
public class Conn {

	static Connection connection;
	static Statement stat;
	static ResultSet rs;

	public Connection getConnection() {
		try {
			Class.forName("com.mysql.jdbc.Driver");
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
		Connection conn=null;
		try {
			conn=DriverManager.getConnection("jdbc:mysql://localhost:3306/mall", "root", "Gepoint");
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return conn;
	}
	
	public static void main(String[] args) {
		Conn c=new Conn();
		connection=c.getConnection();
		try {
			stat=connection.createStatement();
			rs=stat.executeQuery("select * from spbrands limit 10");
			while(rs.next()) {
				String brand=rs.getString("Brand");
				System.out.println("品牌:"+brand);
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}
```

<!--more-->