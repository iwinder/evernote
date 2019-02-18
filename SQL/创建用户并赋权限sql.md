```sql
CREATE SCHEMA sparkredb DEFAULT CHARACTER SET utf8 COLLATE utf8_bin ;

set global validate_password_policy=0;

create user 'sparkredb'@'%' identified by 'manager1';

grant all privileges on sparkredb.* to 'sparkredb'@'%' with grant option;

flush privileges;



grant all privileges on qycmstest.* to 'qycmsdev'@'%' with grant option;
```
