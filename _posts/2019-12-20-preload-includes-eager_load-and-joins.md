---
layout: post
title: "Preload, Includes, Eager_load and Joins"
date: 2019-12-20 22:32:10 +0100
categories: ['Ruby', 'Ruby on Rails']
---

In Rails, we have different methods to load join data. `preload`, `eager_load`, `includes` and `joins`, but what are the difference between them? In fact, we can classify them by `JOIN` 

Based on wether generate a `join` clause, we can separate the 4 methods into two kinds. `preload` and `includes` create **two** SQL query, one for the original table and one for the associated table; while `eager_load` and `joins` create only **one** SQL query, this query uses a `JOIN` clause to associate the two tables.



## Preload

```ruby
User.preload(:posts).to_a

# =>
SELECT "users".* FROM "users"
SELECT "posts".* FROM "posts"  WHERE "posts"."user_id" IN (1)
```

`preload` generate two queries, one for `users` and one for `posts`. But you cannot use the associated table in a `where` clause.

```ruby
User.preload(:posts).where(posts: {id: 1}).to_a

# =>
ActiveRecord::StatementInvalid (SQLite3::SQLException: no such column: posts.id)
SELECT "users".* FROM "users" WHERE "posts"."id" = ? [["id", 1]]
```



## Includes

```ruby
User.includes(:posts).to_a

# =>
SELECT "users".* FROM "users"
SELECT "posts".* FROM "posts" WHERE "posts"."user_id" IN (1)
```

Just like  `preload`, it generate two separate queries. 

However, `includes` is more advanced, depends on the whether a `where` or `order` clause is used for the associated table, a `LEFT OUTER JOIN` or two separete queries will be used.

```ruby
User.includes(:posts).where(id: 1).to_a

#=>
SELECT "users".* FROM "users" WHERE "users"."id" = 1;
SELECT "posts".* FROM "posts" WHERE "posts"."user_id" IN (1);
```

By default, it will use two separate queries, but you can also force it to use `LEFT OUTER JOIN` through `references`.

```ruby
User.includes(:posts).references(:posts).where(id: 1).to_a

#=>
SELECT "users"."id" AS t0_r0, "users"."created_at" AS t0_r1, "users"."updated_at" AS t0_r2, "posts"."id" AS t1_r0, "posts"."body" AS t1_r1, "posts"."user_id" AS t1_r2, "posts"."created_at" AS t1_r3, "posts"."updated_at" AS t1_r4 
FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id" WHERE "users"."id" = 1;
```



## Eager_load

```ruby
User.eager_load(:posts).to_a

#=>
SELECT "users"."id" AS t0_r0, "users"."created_at" AS t0_r1, "users"."updated_at" AS t0_r2, "posts"."id" AS t1_r0, "posts"."body" AS t1_r1, "posts"."user_id" AS t1_r2, "posts"."created_at" AS t1_r3, "posts"."updated_at" AS t1_r4 
FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
```

`eager_load` genereate a single SQL query with `LEFT OUTER JOIN`.



## Joins

```ruby
User.joins(:posts).to_a

#=>
SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
```

`joins` genereate a single SQL query with `INNER JOIN`.

Be careful when you use `joins`, because `joins` will cause `N + 1` problem.

```ruby
User.joins(:posts).map {|user| user.posts.size}

#=>
SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
SELECT COUNT(*) FROM "posts" WHERE "posts"."user_id" = ?  [["user_id", 1]]
SELECT COUNT(*) FROM "posts" WHERE "posts"."user_id" = ?  [["user_id", 2]]
SELECT COUNT(*) FROM "posts" WHERE "posts"."user_id" = ?  [["user_id", 3]]
...
```
