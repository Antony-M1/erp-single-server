# Summary

This page mainly taks about how to handle the DB effectively and handle the Data in better way with proper resource.


## User Story 1
I have worked in the database the size is `780+ MB` the main reason is we are tracking a live location log, which is `2.2+ crore` data is there that's the main reason. We are thinking to use the DB effectively and assign the 4GB of `innodb_buffer_pool_size=4G`.

The reason is we want to run the DB effectively and freely.


## Checks.

### Check the Ram usage.
This check is mainly used to check the how much i assigned how much its used currently in DB 

```sql
SELECT 
  @@innodb_buffer_pool_size / 1024 / 1024 AS configured_mb,
  (SELECT variable_value FROM information_schema.global_status 
   WHERE variable_name = 'Innodb_buffer_pool_pages_total') * 16 / 1024 AS total_pages_mb,
  (SELECT variable_value FROM information_schema.global_status 
   WHERE variable_name = 'Innodb_buffer_pool_pages_data') * 16 / 1024 AS used_by_data_mb,
  (SELECT variable_value FROM information_schema.global_status 
   WHERE variable_name = 'Innodb_buffer_pool_pages_dirty') * 16 / 1024 AS dirty_pages_mb,
  (SELECT variable_value FROM information_schema.global_status 
   WHERE variable_name = 'Innodb_buffer_pool_pages_free') * 16 / 1024 AS free_mb;
```
**output**

```
+---------------+----------------+-----------------+----------------+-------------+
| configured_mb | total_pages_mb | used_by_data_mb | dirty_pages_mb | free_mb     |
+---------------+----------------+-----------------+----------------+-------------+
| 4096.00000000 |           4056 |       782.03125 |       1.484375 | 3273.890625 |
+---------------+----------------+-----------------+----------------+-------------+
```
