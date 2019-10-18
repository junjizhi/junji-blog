---
layout: post
title: "How To Set Default Value: Postgres vs. MySQL"
categories: [All, Technical]
tags: [sql, postgres, mysql]
fullview: false
excerpt: Compare the syntax of setting default values in Postgres and MySQL
comments: true
---

## Default to current timestamp (It works in Postgres and MySQL)
```sql
create table t (
  id int,
  created_at timestamp default current_timestamp
);

insert into t(id) values(1);
```

Result:

```sql
select * from t;

 id |         created_at         
----+----------------------------
  1 | 2020-05-18 23:28:27.121457
(1 row)
```

### Postgres specific syntax
```sql
create table t (
  id int,
  created_at timestamp default now()
);
```
This is because Postgres supports both function or constant as default values. So both `now()` and `current_timestamp`
are legit values. But MySQL doesn't.

## Auto increment

### Postgres

Postgres has a speical type [`serial`](https://www.postgresql.org/docs/9.1/datatype-numeric.html#DATATYPE-SERIAL) for this purpose:
```sql
 create table t (
  id serial,
  created_at timestamp default now()
);
```

### MySQL

MySQL requires `auto_increment` to be a key:
```sql
create table t(
  id int auto_increment,
  created_at timestamp default current_timestamp,
  primary key(id)
);
```

## Change default value of an existing column

### Postgres

Set default timestamp to one hour from now (It's where Postgres Syntax shines):

```sql
alter table only t alter column created_at set default now() + interval '1 hour';
```

Remove default value completely:

```sql
alter table only t alter column created_at drop default;
```

### MySQL

Remove default value:

```sql
alter table t modify column created_at timestamp;
```

Notice the difference with Postgres syntax in `alter column` vs `modify column` parts.

MySQL default value has to be a constant, so we can't do the now plus interval easily in MySQL. We may implement the same function with [triggers](https://stackoverflow.com/a/7024612/8175889).

## Good Resources
Above only scratches the surfaces of default values. Below are some links I found useful to dig deeper.

- [MySQL data types](https://dev.mysql.com/doc/refman/8.0/en/data-type-defaults.html)
- [PostgreSQL CREATE TABLE](https://www.postgresql.org/docs/8.2/sql-createtable.html#AEN48910) (See `default_expr` section)
- [StackOverflow](https://stackoverflow.com/a/10603198/8175889) 
- [Postgres vs. MySQL](https://www.postgresqltutorial.com/postgresql-vs-mysql/)

## Summary
This post summarizes two uses of SQL default values: current timestamp and auto increment. 

There are some minor syntax differences between Postgres and MySQL. Postgres syntax is a bit more flexible because it supports using functions as default value.

If you want to learn more, sign up for my SQL course at [BackToSQL.com](https://backtosql.com/).

Thank you for your read!