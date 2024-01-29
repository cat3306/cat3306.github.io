# MySQL8.0及以上 第三方客户端连接出现 Authentication plugin 'caching_sha2_password' cannot be loaded


<!--more-->

原因是mysql 8.0之前的版本加密规则是mysql_native_password，在8.0之后加密规则是caching_sha2_password。解决办法是一种升级第三方客户端，第二种是把mysql用户登录密码加密规则还原成mysql_native_password

- 第一步先登录mysql 

  ````bash
  $ mysql -u root -p
  #输入密码
  ````

- 修改账户密码的加密规则并更新用户密码

  ````sql
    #修改加密规则 
  ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
  
   #更新一下用户的密码 
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';  
  ````

- 刷新权限

  ```sql
  FLUSH PRIVILEGES; 
  ```

- 重置密码

  ````sql
  alter user 'root'@'localhost' identified by 'pwd';
  ````

  

