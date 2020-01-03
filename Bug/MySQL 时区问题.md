# MySQL 时区错误
最近在使用 Mybatis 和 MySQL 开发的过程中遇到个奇怪的问题，经过排查发现是 JDBC driver 的问题，报错如下
```
java.sql.SQLException: The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```

## 解决方法

在配置mysql时，加上 serverTimezone=UTC

`jdbc:mysql://127.0.0.1:3306/Springcloud_Sell?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC`