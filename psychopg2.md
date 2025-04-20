# Knowledge needed for COMP9120 Assignment2
## Python Database API
Python’s Database API communicates with database systems supporting SQL
- Specific modules for each db engine (eg: Oracle, Postgres, IBM DB2, etc) provide an implementation for common DB-API functionality
## Model for communicating with the database
- Acquire a `connection`
- Create a `cursor` object
- Execute queries using the `cursor` object to __send__ queries and __fetch__ results
- _Exception mechanism_ to handle errors
- Release resource!
### psycopg2: communicate with database 
- communicate with the database
```Python
import psycopg2
try:
    # fetch connection object to connect to the database
    conn = psycopg2.connect(database="postgres",
        user="test",
        password="secret",
        host="host")
    # fetch cursor prepare to query the database
    curs = conn.cursor()
    # execute a SQL query
    curs.execute("""
        SELECT name
        FROM Student NATURAL JOIN Enrolled
        WHERE uos_code = ‘COMP9120’
        """)
    # can loop through the resultset
    for result in curs:
        
    # for illustrating close methods – calling close() on cursor and connection
    # objects will release their associated resources.
    curs.close()
    conn.close()
except Exception as e: # error handling
    print("SQL error: unable to connect to database or execute query")
    print(e)
```
## Cursor methods
- `execute(operation[,parameters])`
    - Execute a query or a command with the SQL provided as the operation argument
    - the SQL query parameter values provided in the parameters argument
- `executemany(operation[,parameters])`
    - Can execute the same query(operation) multiple times, each time with a _different_ parameter set
- `fetchone()`
    - Fetch the next row of a query result set, or returns `None` if no more data is available
    - A `ProgrammingError` is raised if the previous call to `execute*()` __did not__ produce any result set or __no call__ was issued yet
- `fetchmany([size=cursor.arraysize])`
    - Return multiple rows from the result set for a given query, where one can specify the desired number of rows to return (where they are available)
- `fetchall()`
    - Fetch all _remaining_ rows
- `callproc(procname[, parameters])`
    - [Calling Stored Procedures from Python DB-API](#calling-stored-procedures-from-python-db-api)
- `close()`
    - release
## Error Handling
- `Connection` May fail due to server/connection problems
- SQL _query_ could time out
- _Constraint_ violation
### psycopg2: Handling Errors
- `Warning` raised for warnings such as data truncation on insert, etc
- `Error` raised for various db-related errors

    - __psycopg2.Error__:  detailed SQL error codes and messages

        - `pgerror` string of the error message returned by backend
        - `pgcode` string with the SQLSTATE error code returned by backend
- Example：
```Python
try:
    psycopg2.connect(database="postgres",
        user="test",
        password="secret",
        host="host")
except psycopg2.Error as e:
    print("Problem connecting to database:")
    print(e.pgerror)
    print(e.pgcode)
```
## Avoiding SQL Injection
### NEVER 
- __NEVER ever use Python string concatenation (+) or string parameter interpolation (%) to pass variables to a SQL query string!__
    - ~~`'WHERE password =' + pwd'`~~
    - ~~`WHERE password = {}`.format(pwd)~~
    - ~~f`WHERE password = {pwd}`~~
### Passing query parameters
- Anonymous parameters: pass `Tuple` or `List`
    ```Python
    SQL = "INSERT INTO authors (name) VALUES (%s)"  # Note: no quotes
    data = ("O'Reilly", )
    cur.execute(SQL, data)  # Note: no % operator
    ```
- Named parameters: pass `Dictionary`

    ```Python
    cur.execute("""
        INSERT INTO some_table (id, created_at, updated_at, last_name)
        VALUES (%(id)s, %(created)s, %(created)s, %(name)s);
            """,
        {'id': 10, 'name': "O'Reilly", 'created': datetime.date(2020, 11, 18)})
    ```
- Pass in table names: functionalities of `psycopg.sql`
    ```python
    cur.execute(
    SQL("INSERT INTO {} VALUES (%s)").format(Identifier('numbers')),
    (10,))
    ```
> [Psycopg 2.9.10 documentation: Passing parameters to SQL queries](https://www.psycopg.org/docs/usage.html#passing-parameters-to-sql-queries)
## Stored Procedures (Stored Functions)
### Features
- Stored Procedures can have parameters
    - parameter types must match
    - three different modes
        - `IN` arguments to procedure
        - `OUT` return values
        - `INOUT` combination of `IN` and `OUT`
### Calling Stored Procedures from Python DB-API
- `Cursor` objects have an explicit `callproc()` method
    - Makes the `OUT` parameters as resultset available through the standard `fetch*()` methods
- Calling a stored procedure with `IN/OUT` parameter:
    ```Python
    curs.callproc(“RateStudent_INOUT”, [101, "comp9120"]) # no out parameter here
    output = curs.fetchone()
    result = output[0]
    ```
### PL/pgSQL document function
- [Calling function](https://www.postgresql.org/docs/current/sql-syntax-calling-funcs.html)
- [Aggregate Functions](https://www.postgresql.org/docs/9.5/functions-aggregate.html)
- [Structure of PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql-structure.html)
- [Control Structures](https://www.postgresql.org/docs/7.2/plpgsql-control-structures.html)