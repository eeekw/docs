### 连接错误

```json
{
  code: 'ER_NOT_SUPPORTED_AUTH_MODE',
  errno: 1251,
  sqlMessage: 'Client does not support authentication protocol requested by server; consider upgrading MySQL client',
  sqlState: '08004',
  fatal: true
}
```
解决方法：
```bash
mysql -u root -p
alter user 'root'@'localhost' identified with mysql_native_password by '12345678';
flush privileges;
```