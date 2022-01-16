---
Author: Joe Chen
Published: 2021-12-07
Tags: [CY2021, Open Source, Go, Database]
---

# ROD 1 - dbutil: An ergonomic SQL query builder

## The history

Before I started programing web applications in Go, [Microsoft Access](https://www.microsoft.com/en-us/microsoft-365/access) was the most advanced database system I had used with ASP (not .NET, and no kidding, I made a backend that scaled to 12k users with this combination in 2011).

[Go Walker](https://github.com/unknwon/gowalker) was the first ever web application I wrote that involved using ORM-like framework named [Qbs](https://github.com/coocood/qbs) (later migrated to [XORM](https://github.com/go-xorm/xorm)). I took the first and the only SQL class in college after I implemented the initial version of database layer in [Gogs](https://github.com/gogs/gogs) (thanks to XORM).

Since then, I had been big fan of using ORMs for interacting with databases in Go, doesn't matter if the application actually needs to be database-agnostic.

## The problem

Speaking of ORMs in Go, they are (only) convenient for CRUD applications.

I used to be obsessed with automagic side effects when I first started working with database and ORMs in Go. However, the more codebases I've been working on and the more people I've been collaborating with, the more I enjoy with the explicitness of the code I write and review.

I heard ActiveRecord in Ruby is awesome, but I do not like nor have used Ruby so I can't comment. But from my experiences so far with ORMs in Go, I often start with good impressions and then being annoyed, disappointed and in the end, outrageous when try to deal with subtle details. I feel so upset that existing Go ORMs are lacking the spirit of craftsmanship (a general problem), not to mention the absence of design principles (not relevant to this ROD).

There are many things I can complaint about ORMs in Go, but just to name few major ones:
- No practical migration solution.
- Opaque behaviors and hard to predict what would happen.
- Insufficient code comments and docstrings.
- Being over-smart that not fine-touching simple things nor doing great job for complex tasks.
- Little efforts being put into making good abstractions.
- Logging is kinda standard thing to have, but with terrible UX.
- Lack of metrics, logs, and traces for observability.

Then I started to wonder, should I stop using ORMs in Go?

## The challenge

Ever since I started working at [Sourcegraph](https://about.sourcegraph.com/), where we use pure SQLs to interact with [PostgreSQL](https://www.postgresql.org/), I've fall in love with this approach. SQLs do exactly what we instruct, nothing less and nothing more.

The bad news though, is that writing pure SQLs is only nice for applications with single database backend. It would feel somewhat stupid to write pure SQLs for database-agnostic applications. At least for me, I can't imagine maintaining multiple versions of SQLs all over the places in my database layer code.

What about database schema migrations?
- This is a solved problem for applications with single database backend, [golang-migrate](https://github.com/golang-migrate/migrate) is the one I have used with some level of success.
- This is not solved with Go ORMs, in any practical terms (I'm talking about versioned schema changes, upgrade and downgrade).

## The attempt

After some brainstorming, I realized what I need is really just a SQL query builder, to write database-agnostic applications while not having to maintain multiple versions of SQLs. The semantics of the SQL query builder should be close enough to native SQLs but taking care of nuance for different database backends.

The [upper.io/v4](https://github.com/upper/db) is the most promising package that I found can be used purely as a SQL query builder.

As for migrations, [people have asked the same](https://github.com/upper/db/issues/248) which directed me to the [goose](https://github.com/pressly/goose) project, it supports both SQLs and Go functions or mixes of both as migration scripts.

Database-agnostic migrations lives as a dream for a bit longer, and will be researched in a separate ROD. I may take some inspirations from how [GORM](https://github.com/go-gorm) handles it.

## The vision

Building an ergonomic toolkit for building database-agnostic applications, with great attention to UX from general usage to subtle details.

First-class support of OpenTelemetry and Go stdlib tracing, and encode application name to identify the source of connections for logging.

## The plan

- Start building on top of upper.io/v4, and only support PostgreSQL, MySQL and SQLite.
- First experiment with `unknwon.dev/orbiter`.
- I may end up forking the upper.io/v4 or writing a new SQL query builder from scratch in future iterations to accomplish the vision.
  - As upper.io/v4 does not handle [conflicting column names in joins.](https://github.com/upper/db/issues/533)
- Ultimately, this is just a small step for refactoring the database layer of the Gogs project. Before getting there, extra RODs are likely needed for sister projects such as `dbmigrate` and `dbtesting` for database-agnostic migrations and testings respectively.

## Follow ups

- 2021-12-09: Due to `dbutil` is such a common name for database utilities and exists in many applications, I decided to pick a different and unique name [`norm`](https://github.com/go-norm/norm) to avoid import path conflicts. This allows applications to continue using `dbutil` as the internal wrapper around `norm` for sophisticated desires.
