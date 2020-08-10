---
title: 【JAVA】利用java针对Mysql进行封装成（jdbc）工具类Dbutil完整实现（增删改查注册查重，适合初学者，附源码）
date: 2019-10-11 10:29:05
tags:
  - Mysql
  - javaWeb
  - JDBC
  - 工具类
top: true
img: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1570772619423&di=57d8af8229fdee4f5f30c12d80b234b0&imgtype=0&src=http%3A%2F%2Fwww.javacui.com%2Fueditor%2Fphp%2Fupload%2Fimage%2F20140818%2F1408329495178606.gif
categories: javaWeb
---
## 写在前面
搭建博客，没有好的博文，把刚入门的博文整理一下。找了一些关于封装jdbc工具类的帖子，总结经验学习封装一个工具类供初学者学习。

## 主要内容包括以下几点：
也就适合自己学习学习

1、private Connection getConnection(String url)  连接数据库，url为数据库名，方便连接。

2、private PreparedStatement setPs(String url, String sql)  加载sql语句 url同样内容。

3、private void CloseAll() 关流。

4、public List<Map<String, Object>> select(String url, String sql, Object[] obj) 查询功能，Object[] obj 充当占位符?。

5、public int update(String url, String sql, Object[] obj) 添加、修改、删除功能，obj同上。

6、public int register(String url, String sql,String sql1, Object[] obj,Object[] obj1) 注册功能 判断数据库是否存在相同数据

## 代码如下:
刚入门java初学者可以借鉴，没有用配置数据库连接属性文件，毕竟初学者不会哈哈
### JDBC工具类
``` java
package com.Dbutil;

import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author admin
 * @创建时间 2018年7月21日
 */
public class Dbutil {
	// 数据库用户名
	private static final String USERNAME = "root";
	// 数据库密码
	private static final String PASSWORD = "root";
	// 驱动信息
	private static final String DRIVER = "com.mysql.jdbc.Driver";
	// 数据库地址
	private static final String URL = "jdbc:mysql://localhost:3306/";

	private Connection con = null;
	private PreparedStatement ps = null;
	private ResultSet rs = null;
	private ResultSetMetaData rst = null;

	// 静态加载驱动
	static {
		try {
			Class.forName(DRIVER);
			System.out.println("数据库驱动加载成功！");
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println("数据库驱动加载异常！");
		}
	}

	// 连接数据库(url数据库为数据库名)
	private Connection getConnection(String url) {
		try {
			con = DriverManager.getConnection(URL + url, USERNAME, PASSWORD);
			System.out.println("数据库连接成功！");
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println("数据库连接异常！");
		}
		return con;
	}

	// 加载sql语句
	private PreparedStatement setPs(String url, String sql) {
		getConnection(url);
		try {
			// 获得连接
			ps = con.prepareStatement(sql);
			System.out.println("加载sql语句成功!");
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println("加载sql语句成功异常！");
		}
		return ps;
	}
	
	//关流
	private void CloseAll() {
		if(rs!=null) {
			try {
				rs.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				System.out.println("查询结果集关闭失败！");
				e.printStackTrace();
			}
		}
		if(ps!=null) {
			try {
				ps.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				System.out.println("语句对象关闭异常！");
				e.printStackTrace();
			}
		}
		if(con!=null) {
			try {			
				con.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				System.out.println("语句对象关闭异常！");
				e.printStackTrace();
			}
		}
	}
	
	//查询 (Object[] obj 充当占位符? )
	public List<Map<String, Object>> select(String url, String sql, Object[] obj) {
		List<Map<String, Object>> list = new ArrayList<Map<String, Object>>();
		setPs(url, sql);
		try {
			if (obj != null) {
				//对占位符？进行复制
				for (int i = 0; i < obj.length; i++) {
					ps.setObject(i + 1, obj[i]);//从第一个开始 i+1 ，值为obj[i]
				}
			}
			// 执行
			rs = ps.executeQuery();
			rst = rs.getMetaData();
			while (rs.next()) {
				Map<String, Object> map = new HashMap<String, Object>();
				for (int i = 1; i <= rst.getColumnCount(); i++) {
					String key = rst.getColumnName(i);
					map.put(key, rs.getString(key));
				}
				list.add(map);
			}
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			System.out.println("查询出现异常！");
			e.printStackTrace();
		} finally {
			CloseAll();
		}
		return list;
	}
	
	//查询无set (用于展示所有列表)
	public List<Map<String, Object>> select(String url, String sql) {
		return select(url,sql,new Object[] {}); 
	}
	
	//添加、删除、修改（）
	public int update(String url, String sql, Object[] obj) {
		ps= setPs(url, sql);
		try {
			if (obj != null) {
				for (int i = 0; i < obj.length; i++) {
					ps.setObject(i + 1, obj[i]);
				}
			}
			return ps.executeUpdate();
		} catch (SQLException e) {
			// TODO: handle exception
			System.out.println("更新失败,请检查！");
			e.printStackTrace();
		}finally {
			CloseAll();
		}
		return -1;
	}
	
	//注册
	public int register(String url, String sql,String sql1, Object[] obj,Object[] obj1) {
		List<Map<String, Object>> list = select(url, sql1, obj1);	
		if(list.size()==0) {//判断是否有数据
			ps=setPs(url, sql);
			try {
				if (obj != null) {
					for (int i = 0; i < obj.length; i++) {
						ps.setObject(i + 1, obj[i]);
					}
				}				
					return ps.executeUpdate();	
			} catch (SQLException e) {
				// TODO: handle exception
				System.out.println("更新失败,请检查！");
				e.printStackTrace();
			}finally {
				CloseAll();
			}
		}
		
		return -1;
	}
	
}
``` 
### 补充说明:

操作数据库的一般性步骤如下：

（1）连接数据库,加载驱动: Class.forName(DRIVER); DRIVER = "com.mysql.jdbc.Driver";

（2）利用用户名和密码及数据库的名字连接，这一步才是真正的连接:

connection = DriverManager.getConnection(URL+url, USERNAME, PASSWORD); 

其中：String URL = "jdbc:mysql://localhost:3306/url"，url为数据库名，传参方便使用;

（3）编写一个sql语句，其中的参数用?来代替，然后将参数写到Object[]里。

执行:ps = connection.prepareStatement(sql); 然后将参数从Object[] obj里取出来填充到ps里。

（4）如果是增、删、改执行:rs = ps.executeUpdate(); 其中的rs是执行完影响的数据库里的行数，也即几条记录。如果是查询执行:resultSet = pstmt.executeQuery(); 返回的类型是ResultSet类型。之后就是把resultSet 弄成Map或List<Map>传递出去，给查询者看。

（5）关于查询操作，在得到resultSet后利用getMetaData得到表的结构信息，如getColumnCount()得到有多少个列。String key = rst.getColumnName(i); 得到每个列的属性名称，如是id、username还是pswd.然后放到Map、List<Map>里。

（6）关于注册操作，类似于增，但在新增之前先进行查询有无存在，若存在不新增，不存在再进行新增操作，即先调用查询方法进行先查询，再新增。

### Servlet部分代码如下（登录）：

```java
package com.servlet;

import java.io.IOException;
import java.util.List;
import java.util.Map;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import com.Dbutil.Dbutil;

/**
 * Servlet implementation class LoginServlet
 */
@WebServlet("/LoginServlet")
public class LoginServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       

	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String user = request.getParameter("user");
		String pwd = request.getParameter("pwd");
		Dbutil jdbc=new Dbutil();
		if(user.length()==0||pwd.length()==0) {
			request.setAttribute("session", "用户密码不能为空!");
			request.getRequestDispatcher("index.jsp").forward(request, response);
		}else {
			List<Map<String, Object>> list = jdbc.select("manager", "select username,password from user where username=? and password=?", new Object[] {user,pwd});
			if(list.size()!=0) {
				HttpSession session = request.getSession();
				session.setAttribute("welcome", user);
				request.getRequestDispatcher("SelectAllServlet").forward(request, response);
				System.out.println("登陆成功");
			}else {
				request.setAttribute("session", "账号密码错误！");
				request.getRequestDispatcher("index.jsp").forward(request, response);
			}
		}
		
	}

}
```
### 注册：
```java
package com.servlet;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.Dbutil.Dbutil;

/**
 * Servlet implementation class RegisterServlet
 */
@WebServlet("/RegisterServlet")
public class RegisterServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String username=request.getParameter("username");
		String pwd=request.getParameter("pwd");
		String pwdn = request.getParameter("pwdn");
		Dbutil jdbc=new Dbutil();
		if(username.length()==0||pwd.length()==0||pwdn.length()==0) {
			request.setAttribute("mession", "各项数据不能为空");
			request.getRequestDispatcher("register.jsp").forward(request, response);
		}else{
			int i = jdbc.register("manager", "insert into user values(null,?,?)","select username from user where username=?", new Object[] {username,pwd},new Object[] {username});
			if(pwd.equals(pwdn)) {
				if(i>0) {
					request.setAttribute("session", "注册成功");
					request.getRequestDispatcher("index.jsp").forward(request, response);
				}else {
					request.setAttribute("mession", "用户名重复，注册失败");
					request.getRequestDispatcher("register.jsp").forward(request, response);
				}
			}else {
				request.setAttribute("mession", "两次密码输入不正确");
				request.getRequestDispatcher("register.jsp").forward(request, response);
			}
		}
	}

}
```
### 查询：
```java
package com.servlet;

import java.io.IOException;
import java.util.List;
import java.util.Map;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.Dbutil.Dbutil;

/**
 * Servlet implementation class SelectAllServlet
 */
@WebServlet("/SelectAllServlet")
public class SelectAllServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       


	/**
	 * @see HttpServlet#service(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		Dbutil jdbc=new Dbutil();
		List<Map<String, Object>> list = jdbc.select("manager", "select * from emp");
		request.setAttribute("list", list);
		request.getRequestDispatcher("list.jsp").forward(request, response);
	}

}
```

加入了一些判断，其余的类似一样的调用即可。

其余的就不写了，偷个懒！