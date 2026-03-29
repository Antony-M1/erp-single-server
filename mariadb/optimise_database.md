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

