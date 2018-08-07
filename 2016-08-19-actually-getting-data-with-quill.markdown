---
layout: post
title: "Actually getting data with quill"
date: 2016-08-19 19:47:45 +0200
comments: true
categories: 
 - Scala
 - programming
 - open-source
 - MYSQL
---

[Quill](http://getquill.io) is a pretty cool scala library for generating SQL quieries at compile time. It has beautiful syntax and I really enjoy using it BUT it is *really* difficult to get set up with. This is a simple tutorial that should get you to the point where you have returned data from a MYSQL database.
<!-- more -->

To start with we need to download the library. I will use SBT to include the library with the following code:
```scala
libraryDependencies ++= Seq(
  "mysql" % "mysql-connector-java" % "5.1.38",
  "io.getquill" %% "quill-jdbc" % "0.8.0"
)
```
This will download the MYSQL driver needed as well ast the main quill library. Now for some code. All of this will be in the `src/main/scala` directory. The first file needed is `application.properties` which will contain the needed application settings:
```YAML
ctx.dataSourceClassName=com.mysql.jdbc.jdbc2.optional.MysqlDataSource
ctx.dataSource.url=jdbc:mysql://127.0.01/TestDB
ctx.dataSource.user=root
ctx.dataSource.password=passw0rd
ctx.dataSource.cachePrepStmts=true
ctx.dataSource.prepStmtCacheSize=250
ctx.dataSource.prepStmtCacheSqlLimit=2048
ctx.connectionTimeout=30000
```
This is all the configuration we need to connect to a MYSQL database hosted on localhost and the database TestDB. Quill allows easy object with case classes so we need to create these to hold the data. In this schema I will represent the current US presidential election with a table for candidates:

|\| id (Int) \|| name (String) |\| election(Int) |\| party(Int) |\|
|-------|------------:|------------:|---------:|
| 1 | Hillary Clinton|1 | 1 |
| 2 | Donald Trump | 1 | 2 |

This is a very simple table and we will call it `Candidate`. A case class is all we need to represent this in scala. This will be what quill returns when it gets data from the database. I will put this in a package called DBObjects to store these case classes.

```scala
package com.vogonjeltz.election.db.DBObjects

case class Candidate(id: Int, name: String, election: Int, party: Int) {}
```

Now we need to connect to the database by creating a `JdbcContext`. This requires us selecting our dialect and a naming strategy. The dialect is the type of sql we are usig which in this case is is the `MySQLDialect` and the naming strategy is how the names of the tables and fields are transformed. I will be using Literal which wont transform them at all. The easiest place to put this context is in a package object. I will put it in the db package. Here is `src/main/scala/com/vogonjeltz/package.scala`:

```scala
package com.vogonjeltz.election

import io.getquill.{JdbcContext, Literal, MySQLDialect}

package object db {

  type JdbcDatabase = JdbcContext[MySQLDialect, Literal]

  lazy val ctx = new JdbcDatabase("ctx")

}
```

Now to select data from the database. The methods we need are in that context instance so accessing them is east as importing `db.ctx._`. The `quote` method is what quill uses to create the SQL at compile time (or runtime if it can't figure it out). Here is a test `App`:

```scala
package com.vogonjeltz.election

import com.vogonjeltz.election.db.DBObjects.Candidate
import db._

object Main extends App {

  import ctx._

  val HRCQuery = quote {
    query[Candidate].filter(_.id == 1)
  }

}
```

This will generate the SQL `SELECT * FROM Candidate WHERE id = 1` (or equivalent).  This is can then be executed by running it against the context:

```scala
val HRC = ctx.run(HRCQuery)

println(HRC)
```

This will print out: `Candidate(1, "Hillary Clinton", 1, 1)` and there you go. You have successfully got data from a MYSQL (or whichever db you used).You can read about more complicated queries [here](http://getquill.io).

Thanks for reading,

Freddie
