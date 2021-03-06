# QL

The `ql` adapter for [QL][1] wraps the `github.com/cznic/ql/ql` driver
written by [Jan Mercl][1].

![Note](https://github.com/LizGoro90/db-tour/tree/master/static/img)
> Here you’ll learn about the particularities of the [QL][1] adapter. Before starting to read this detailed information, it is advisable that you take a look at the [getting started](https://upper.io/db.v3/getting-started) page so you become acquainted with the basics of upper-db and you can grasp concepts better.

## Installation

Use `go get` to download and install the adapter:

```go
go get upper.io/db.v3/ql
```

## Setup
### Database Session

Import the `upper.io/db.v3/ql` package into your application:

```go
// main.go
package main

import (
  "upper.io/db.v3/ql"
)
```

Define the `ql.ConnectionURL{}` struct:

```go
// ConnectionURL defines the DSN attributes.
type ConnectionURL struct {
  Database string
  Options  map[string]string
}
```

Pass the `ql.ConnectionURL` value as argument to `ql.Open()` so the `ql.Database` session is created.

```go
settings = ql.ConnectionURL{
  ...
}

sess, err = ql.Open(settings)
...
```

![Note](https://github.com/LizGoro90/db-tour/tree/master/static/img)
> The `ql.ParseURL()` function is also provided in case you need to convert the DSN into a `ql.ConnectionURL`:

```go
// ParseURL parses a DSN into a ConnectionURL struct.
ql.ParseURL(dsn string) (ConnectionURL, error)
```

## Common Database Operations

Once the connection is established, you can start performing operations on the database.

### Example

In the following example, a table named ‘birthday’ consisting of two columns (‘name’ and ‘born’) will be created. Before starting, the table will be searched in the database and, in the event it already exists, it will be removed. Then, three rows will be inserted into the table and checked for accuracy. To this end, the database will be queried and the matches (insertions) will be printed to standard output.

![Note](https://github.com/LizGoro90/db-tour/tree/master/static/img)
> The database operations described above refer to an advanced use of upper-db, hence
they do not follow the exact same patterns of the [tour](https://tour.upper.io/welcome/01) and [getting started](https://upper.io/db.v3/getting-started) page.

The `birthday` table with the `name` and `born` columns is created with these SQL statements:

```sql
--' example.sql
DROP TABLE IF EXISTS birthday;

CREATE TABLE birthday (
  name string,
  born time
);
```

The `ql` command line tool is used to create an `example.db` database file:

```
rm -f example.db
cat example.sql | ql -db example.db
```

The rows are inserted into the `birthday` table. The database is queried for the insertions and is set to print them to standard output.

```go
// example.go

package main

import (
  "fmt"
  "log"
  "time"
  "upper.io/db.v3/ql"
)

var settings = ql.ConnectionURL{
  Database: `example.db`, // Path to database file
}

type Birthday struct {
  // The 'name' column of the 'birthday' table
  // is mapped to the 'name' property.
  Name string `db:"name"`
  // The 'born' column of the 'birthday' table
  // is mapped to the 'born' property.
  Born time.Time `db:"born"`
}

func main() {

  // Attempt to open the 'example.db' database file
  sess, err := ql.Open(settings)
  if err != nil {
    log.Fatalf("db.Open(): %q\n", err)
  }
  defer sess.Close() // Closing the session is a good practice.

  // The 'birthday' table is referenced.
  birthdayCollection := sess.Collection("birthday")

  // Any rows that might have been added between the creation of
  // the table and the execution of this function are removed. 
  err = birthdayCollection.Truncate()
  if err != nil {
    log.Fatalf("Truncate(): %q\n", err)
  }

  // Three rows are inserted into the 'Birthday' table.
  birthdayCollection.Insert(Birthday{
    Name: "Hayao Miyazaki",
    Born: time.Date(1941, time.January, 5, 0, 0, 0, 0, time.Local),
  })

  birthdayCollection.Insert(Birthday{
    Name: "Nobuo Uematsu",
    Born: time.Date(1959, time.March, 21, 0, 0, 0, 0, time.Local),
  })

  birthdayCollection.Insert(Birthday{
    Name: "Hironobu Sakaguchi",
    Born: time.Date(1962, time.November, 25, 0, 0, 0, 0, time.Local),
  })

  // The database is queried for the rows inserted.
  res := birthdayCollection.Find()

  // The 'birthdays' variable is filled with the results found.
  var birthdays []Birthday

  err = res.All(&birthdays)
  if err != nil {
    log.Fatalf("res.All(): %q\n", err)
  }

  // The 'birthdays' variable is printed to stdout.
  for _, birthday := range birthday {
    fmt.Printf("%s was born in %s.\n",
      birthday.Name,
      birthday.Born.Format("January 2, 2006"),
    )
  }
}

```

The Go file is compiled and executed using `go run`:

```
go run example.go
```

The output consists of three rows including names and birthdates:

```
Hayao Miyazaki was born in January 5, 1941.
Nobuo Uematsu was born in March 21, 1959.
Hironobu Sakaguchi was born in November 25, 1962.
```

## Specifications
### SQL Builder

You can use the [query builder](/db.v3/lib/sqlbuilder) for any complex SQL query:

```go
q := b.Select(
    "p.id",
    "p.title AD publication_title",
    "a.name AS artist_name",
  ).From("artists AS a", "publication AS p").
  Where("a.id = p.author_id")

var publications []Publication
if err = q.All(&publications); err != nil {
  log.Fatal(err)
}
```

### Escape Sequences

There might be characters that cannot be typed in the context you're working, or else would have an undesired interpretation. Through `db.Func` you can encode the syntactic entities that cannot be directly represented by the alphabet: 

```go
res = sess.Find().Select(db.Func("DISTINCT", "name"))
```

On the other hand, you can use the `db.Raw` function so a given value is taken literally: 

```go
res = sess.Find().Select(db.Raw("DISTINCT(name)"))
```

![Note](https://github.com/LizGoro90/db-tour/tree/master/static/img)
> `db.Raw` can also be used as a condition argument, similarly to `db.Cond`.

![Note](https://github.com/LizGoro90/db-tour/tree/master/static/img)
> Click [here][4] to keep learning about different database operations that can be executed with upper-db. 


[1]: https://github.com/cznic/ql
[2]: http://golang.org/doc/effective_go.html#blank
[3]: /db.v3/getting-started
[4]: /db.v3/examples
