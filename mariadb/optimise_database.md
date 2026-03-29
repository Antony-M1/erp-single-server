# Summary

For some scenario if database gets bigger and bigger its failed to handle the information that time we have to increase the database size and help the database to process smoothly.

# Error Traceback
```py
Traceback (most recent call last):
  File "apps/frappe/frappe/database/database.py", line 230, in sql
    self._cursor.execute(query, values)
  File "env/lib/python3.10/site-packages/pymysql/cursors.py", line 153, in execute
    result = self._query(query)
  File "env/lib/python3.10/site-packages/pymysql/cursors.py", line 322, in _query
    conn.query(q)
  File "env/lib/python3.10/site-packages/pymysql/connections.py", line 563, in query
    self._affected_rows = self._read_query_result(unbuffered=unbuffered)
  File "env/lib/python3.10/site-packages/pymysql/connections.py", line 825, in _read_query_result
    result.read()
  File "env/lib/python3.10/site-packages/pymysql/connections.py", line 1199, in read
    first_packet = self.connection._read_packet()
  File "env/lib/python3.10/site-packages/pymysql/connections.py", line 775, in _read_packet
    packet.raise_for_error()
  File "env/lib/python3.10/site-packages/pymysql/protocol.py", line 219, in raise_for_error
    err.raise_mysql_exception(self._data)
  File "env/lib/python3.10/site-packages/pymysql/err.py", line 150, in raise_mysql_exception
    raise errorclass(errno, errval)
pymysql.err.OperationalError: (1205, 'Lock wait timeout exceeded; try restarting transaction')

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "apps/frappe/frappe/app.py", line 120, in application
    response = frappe.api.handle(request)
  File "apps/frappe/frappe/api/__init__.py", line 52, in handle
    data = endpoint(**arguments)
  File "apps/frappe/frappe/api/v1.py", line 40, in handle_rpc_call
    return frappe.handler.handle()
  File "apps/frappe/frappe/handler.py", line 53, in handle
    data = execute_cmd(cmd)
  File "apps/frappe/frappe/handler.py", line 86, in execute_cmd
    return frappe.call(method, **frappe.form_dict)
  File "apps/frappe/frappe/__init__.py", line 1758, in call
    return fn(*args, **newargs)
  File "apps/frappe/frappe/utils/typing_validations.py", line 32, in wrapper
    return func(*args, **kwargs)
  File "apps/frappe/frappe/desk/form/save.py", line 39, in savedocs
    doc.save()
  File "apps/frappe/frappe/model/document.py", line 378, in save
    return self._save(*args, **kwargs)
  File "apps/frappe/frappe/model/document.py", line 408, in _save
    self.check_if_latest()
  File "apps/frappe/frappe/model/document.py", line 856, in check_if_latest
    self.load_doc_before_save(raise_exception=True)
  File "apps/frappe/frappe/model/document.py", line 1161, in load_doc_before_save
    self._doc_before_save = frappe.get_doc(self.doctype, self.name, for_update=True)
  File "apps/frappe/frappe/__init__.py", line 1312, in get_doc
    return frappe.model.document.get_doc(*args, **kwargs)
  File "apps/frappe/frappe/model/document.py", line 85, in get_doc
    return controller(*args, **kwargs)
  File "apps/frappe/frappe/model/document.py", line 126, in __init__
    self.load_from_db()
  File "apps/frappe/frappe/model/document.py", line 174, in load_from_db
    d = frappe.db.get_value(
  File "apps/frappe/frappe/database/database.py", line 514, in get_value
    result = self.get_values(
  File "apps/frappe/frappe/database/database.py", line 621, in get_values
    out = self._get_values_from_table(
  File "apps/frappe/frappe/database/database.py", line 894, in _get_values_from_table
    return query.run(as_dict=as_dict, debug=debug, update=update, run=run, pluck=pluck)
  File "apps/frappe/frappe/query_builder/utils.py", line 87, in execute_query
    result = frappe.db.sql(query, params, *args, **kwargs)  # nosemgrep
  File "apps/frappe/frappe/database/database.py", line 239, in sql
    raise frappe.QueryTimeoutError(e) from e
frappe.exceptions.QueryTimeoutError: (1205, 'Lock wait timeout exceeded; try restarting transaction')

```

# Local Mariadb container fix.

For my local-setup im getting this issue i don this fix.

Before get into the code please refere this page and allocate more memory to the swap from your ROM. [allocate_swap_memory](./allocate_swap_memory.md)

Once you allocated the memory we will start this

## MariaDB Configuration Update (Docker)

### Step 1: Check Memory and Swap

```
free -h
swapon --show
```

---

### Step 2: Access Docker Container

```
docker exec -it <CONTAINER_NAME> bash
```

---

### Step 3: Open MariaDB Configuration File

```
cat /etc/mysql/mariadb.conf.d/50-server.cnf
```

---

### Step 4: Install Nano Editor

```
apt-get update
apt-get install nano
```

---

### Step 5: Edit Configuration File

```
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

---

### Step 6: Update/Add Below Fields

```
max_allowed_packet     = 1G
max_connections        = 200

log_slow_query_file    = /var/log/mysql/mariadb-slow.log
log_slow_query_time    = 10
log_slow_verbosity     = query_plan,explain
log-queries-not-using-indexes

innodb_buffer_pool_size = 10G
```


<details>
  <summary>Here the actual file details</summary>

  ```cnf
  #
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see

# this is read by the standalone daemon and embedded servers
[server]

# this is only for the mariadbd daemon
[mariadbd]

#
# * Basic Settings
#

#user                    = mysql
pid-file                = /run/mysqld/mysqld.pid
basedir                 = /usr
#datadir                 = /var/lib/mysql
#tmpdir                  = /tmp

# Broken reverse DNS slows down connections considerably and name resolve is
# safe to skip if there are no "host by domain name" access grants
#skip-name-resolve

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address            = 127.0.0.1

#
# * Fine Tuning
#

#key_buffer_size        = 128M
max_allowed_packet     = 1G
#thread_stack           = 192K
#thread_cache_size      = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
#myisam_recover_options = BACKUP
max_connections        = 200
#table_cache            = 64

#
# * Logging and Replication
#

# Note: The configured log file or its directory need to be created
# and be writable by the mysql user, e.g.:
# $ sudo mkdir -m 2750 /var/log/mysql
# $ sudo chown mysql /var/log/mysql

# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# Recommend only changing this at runtime for short testing periods if needed!
#general_log_file       = /var/log/mysql/mysql.log
#general_log            = 1

# Error logging goes via stdout/stderr, which on systemd systems goes to
# journald.
# Enable this if you want to have error logging into a separate file
#log_error = /var/log/mysql/error.log
# Enable the slow query log to see queries with especially long duration
log_slow_query_file    = /var/log/mysql/mariadb-slow.log
log_slow_query_time    = 10
log_slow_verbosity     = query_plan,explain
log-queries-not-using-indexes
#log_slow_min_examined_row_limit = 1000

# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replica, see README.Debian about other
#       settings you may need to change.
#server-id              = 1
#log_bin                = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
#max_binlog_size        = 100M

#
# * SSL/TLS
#

# For documentation, please read
# https://mariadb.com/kb/en/securing-connections-for-client-and-server/
#ssl-ca = /etc/mysql/cacert.pem
#ssl-cert = /etc/mysql/server-cert.pem
#ssl-key = /etc/mysql/server-key.pem
#require-secure-transport = on

#
# * Character sets
#

# MariaDB default is now utf8 4-byte character set.
# No Debian specific default is required.

#
# * InnoDB
#

# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
# Most important is to give InnoDB 80 % of the system RAM for buffer use:
# https://mariadb.com/kb/en/innodb-system-variables/#innodb_buffer_pool_size
innodb_buffer_pool_size = 10G

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadbd]

# This group is only read by MariaDB-11.8 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don't understand
[mariadb-11.8]
  ```
</details>

---

### Step 7: Save and Exit

* Press `CTRL + O` → Enter
* Press `CTRL + X`

---

### Step 8: Restart Container

```
docker restart <CONTAINER_NAME>
```


