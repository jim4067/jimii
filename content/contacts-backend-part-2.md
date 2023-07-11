+++
title = "Build a Simple Backend API Server with Actix and Diesel 2"
date = 2023-07-12
category = "Prog"
draft=false

[taxonomies]
tags = ["Rust", "Actix", "Diesel"]
+++

In this part we will ad the logic to expose our data to

But first there are some slight changes we are going to me as suggested by Tev from the Rust Developer community Kenya.

1. We are going to add the `ON DELETE CASCADE` constraint to make sure that when a user is deleted, all of it's associated contacts will be deleted too.
2. Add the `NOT NULL` constraint to the `user_id` foreign key on our `contacts` relation.
3. We will make our `username` field unique and disallow duplications so that it can be easier to search for users.

To implement these changes we will create a new migration to show how to state of our db has changed. We will then populate the generated migrations files with the SQL statements. <!-- ! reread this statement and update with the correct wordings-->

```sql

```

The last change we will make, has to do with how we connect to our database. Currently we only ... and we can use a connection pool which is more error resilient. <!-- !update this sentence with the correct  -->

The concept of connection pooling is the process of having a pool of active connections instead of having to open, maintain and close a connection when a request is made to postgres.

To use this feature in our app we will need to enable the [`r2d2`](https://docs.rs/diesel/latest/diesel/r2d2/index.html#:~:text=When%20used%20inside%20a%20pool,the%20connection%20to%20the%20DB.) feature that comes with diesel.

```bash

```

## Actix <a name="actix"></a>

We will use actix to implement the logic for our api,

to install it

<!-- actix -->
<!-- auth -->
<!-- todo - add unique constraint to the username -->
<!-- todo - on cascade constraint -->
<!-- todo - connection pooling -->
<!-- todo - add readme -->
<!-- todo - add a section on error handling -->

In the [next part](../contacts-backend) we will set up everything with actix and get our api ready.

## Further Reading

-   Check out this great write up on [connection pooling and proxies](https://arctype.com/blog/connnection-pooling-postgres/#:~:text=Connection%20pooling%20is%20the%20process,active%20connection%20to%20the%20user.)
-   Check out this [blog by Brook Jeynes](https://medium.com/@jeynesbrook/building-an-api-in-rust-with-rocket-rs-and-diesel-rs-clean-architecture-8f6092ee2606) on building a web application with rocket.
