# In memory SQLite3 Database

```
import sqlite3

c1 = sqlite3.connect("file::memory:?cache=shared")
c1.execute('CREATE TABLE foo (bar, baz)')
c1.execute("INSERT INTO foo VALUES ('spam', 'ham')")
c1.commit()

c2 = sqlite3.connect("file::memory:?cache=shared")
c2 = sqlite3.connect("file::memory:?cache=shared")
list(c2.execute('SELECT * FROM foo'))

c3 = sqlite3.connect('/tmp/sqlite3.db')
c3.execute("ATTACH DATABASE 'file::memory:?cache=shared' AS inmem")
list(c3.execute('SELECT * FROM inmem.foo'))
```

Note:

- It will create a file on hard disk call “file::memory:?cache=shared” <== Temporary database
- If we use only “:memory:” <== nothing create

References
- https://pythontic.com/database/sqlite/create%20table
