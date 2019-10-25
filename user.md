

## 用户权限管理

UDDB基本完全兼容MySQL的用户权限管理功能。和MySQL不一致的是， 用户的密码只能在create user中指定，
不能在grant中指定。如：

UDDB支持：
```
create user 'robert'@'10.10.%' identified by '123qwe';
grant all privileges on *.* to 'robert'@'10.10.%' ;
```
不支持：
```
grant all privileges on *.* to 'robert'@'10.10.%' identified by '123qwe' ;
```
