> [!IMPORTANT] 
> According to MariaDB documentation, using encryption has an overhead of roughly **_3–5%_**

### Encryption Key Management

For key management, there are four options — two of which are commonly used:

- [File Key Management Plugin](https://mariadb.com/kb/en/file-key-management-encryption-plugin/)
- [Hashicorp Key Management Plugin](https://mariadb.com/kb/en/hashicorp-key-management-plugin/)

Here we are going to use the File Key Management Plugin:

`sudo mkdir /etc/mysql/encryption`

`sudo chmod 755 /etc/mysql/encryption`

`sudo su`

`(echo -n "1;" ; openssl rand -hex 32 ) | sudo tee -a /etc/mysql/encryption/keyfile`

`openssl rand -hex 128 > /etc/mysql/encryption/keyfile.key`

`openssl enc -aes-256-cbc -md sha1 -pass file:/etc/mysql/encryption/keyfile.key -in /etc/mysql/encryption/keyfile -out /etc/mysql/encryption/keyfile.enc`

`chmod 644 /etc/mysql/encryption/keyfile.enc`

`chmod 644 /etc/mysql/encryption/keyfile.key`

### Encrypting Data

To encrypt MariaDB database you need to add following config in /etc/mysql/mariadb.conf.d/50-server.cnf 

`sudo vim /etc/mysql/mariadb.conf.d/50-server.cnf`

```Mariadb
[mariadbd]
performance_schema = ON
# File Key Management
plugin_load_add = file_key_management
file_key_management_filename = /etc/mysql/encryption/keyfile.enc
file_key_management_filekey = FILE:/etc/mysql/encryption/keyfile.key
file_key_management_encryption_algorithm = AES_CTR

# InnoDB Encryption
innodb_encrypt_tables = FORCE
innodb_encrypt_temporary_tables = ON
innodb_encrypt_log = ON
innodb_encryption_threads = 4
encrypt_binlog = ON
```

`sudo systemctl restart mariadb`

After these changes every table that you create would be saved encrypted and you can check that by checking /var/lib/mysql/mysql/[table name].ibd file.

> [!WARNING]
> Existing tables would not be encrypted and you should encrypt them manually by running this query:

`ALTER TABLE db.tbl ENCRYPTED = YES;`
