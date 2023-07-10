+++
title = "Build a Simple Backend API Server with Actix and Diesel 1"
date = 2023-07-10
category = "Prog"

[taxonomies]
tags = ["Rust", "Diesel", "Postgres"]
+++

In this article we will implement a backend api server for a contacts application that uses Postgres for persistent data storage.

**NOTE:** I am working on Linux and some of these instructions might not work for Other OSs.

We will use [Actix](https://actix.rs/docs/whatis) - a powerful and pragmatic web framework, [diesel](https://diesel.rs/) - 'a safe & extensible rust ORM' and [Postgres](https://www.postgresql.org/) for persistent data storage.

<!-- more -->

# Table of contents

1. [Introduction](#introduction)
2. [Diesel and Postgres](#diesel-postgres)
    - [our crud db operations](#crud-impl)
    - [Tests for our database](#db-tests)

## Introduction <a name="introduction"></a>

This article assumes that you are familiar with [Rust](rust-lang.org).

If not fret not, If you have build a server using another framework like express before, it will be easy to identify patterns between the two. _P.S - set up your rust dev environment by following [this guide](rust-lang.org/learn/get-started)_

Scaffold and new Rust project using cargo by running, `cargo new contacts-backend` on the terminal.

Some convenient tools, that will make our development easier that you should install include:

-   [cargo watch](https://github.com/watchexec/cargo-watch) - like nodemon. _auto-reload the server when we make changes_
-   [cargo edit](https://github.com/killercup/cargo-edit) - simplify the addition of dependencies.
-   [cargo clippy](https://github.com/rust-lang/rust-clippy) - a rust linter.

```bash
cargo install cargo-edit cargo-watch
```

Finally, here is how we want to model our file structure, following the separation of concern design principle.

We will create these files as we go along and explain what each does.

```
contacts-backend
├── src
│   ├── data
│   │   ├── mod.rs
│   │   ├── user_repository.rs
│   │   └── contact_repository.rs
│   ├── handlers
│   │   ├── mod.rs
│   │   ├── user_handler.rs
│   │   └── contact_handler.rs
│   ├── models
│   │   ├── mod.rs
│   │   ├── user.rs
│   │   └── contact.rs
│   ├── db.rs
│   ├── schema.rs
│   ├── main.rs
│   └── routes.rs
├── .env
├── diesel.toml
├── Cargo.toml
└── Cargo.lock
```

## Diesel and Postgres <a name="diesel-postgres"></a>

For persistent data storage we will use Postgres DB, locally. I have some notes on getting started with it. If you prefer to use an online hosted Database as a Service like [supabase](https://supabase.com/), that is okay too. Whatever floats your boat.

Our simple database will host two relations, **contacts** and **users**. This is how our schema will look like.

<!-- Create a Postgres Database called `contacts_db` -->

**Table: user_profile**

```
user_id (PK): auto-incremented integer
username: String
password: String
email: String
first_name: String
last_name: String
```

**Table: contact**

```
contact_id (Primary Key): (auto-incremented integer)
user_id (Foreign Key): linking the contact to its owner in the Users table
email: string
Phone: string
city: string
country: string
```

Diesel is a Rust ORM. It also comes with a convenient tool to enable us manage our database schemas. [diesel_cli](https://diesel.rs/guides/configuring-diesel-cli.html).

Add the diesel create to our project, with the `postgres` features flag. We will need `dotenvy` crate to enable use work with our `.env` secrets.

```bash
cargo add diesel --features=postgres
cargo add dotenvy
```

Install the diesel-cli to our system.

```bash
 cargo install diesel_cli --no-default-features --features postgres
```

The `--features` flag tell diesel_cli that we will working with postgres as our primary DB.

**NOTE:**
Make sure you have the client libraries for postgres, mysql and sqlite installed

```bash
sudo apt install libpq-dev libmysqlclient-dev libsqlite3-dev
```

Or you might come across an error similar to `note: ld: library not found for ...`.

Let's get started by creating the database url, in our `.env` file. Replace `username` and `password` with your `username` and `password`. If you don't know about postgres role, check out my [postgres notes I](../postgres-cheats-i).

```bash
echo DATABASE_URL=postgres://username:password@localhost/contacts_db > .env
```

**NOTE:**
Make sure that you don't have any trailing white space for the `DATABASE_URL` variable as dotenvy will panic with this cryptic message `environment variable not found`.

| **❌ 🙅‍♂️**                                        |
| :----------------------------------------------- |
| Don't forget to add .env, to the .gitignore file |

The `setup` command below from diesel, will create our database if we haven't already and an empty `migrations` directory.

```
diesel setup
```

Next, we will create the migrations files for our users relation.

```
diesel migration generate create_users
```

```bash
Output
Creating migrations/2023-07-08-133307_create_users/up.sql
Creating migrations/2023-07-08-133307_create_users/down.sql
```

We will write the SQL for the migration and for reverting the migration. Database migrations become very important whe you are working with a team and decide to change your schema. Read more about why migrations are important in this [Prisma guide](https://www.prisma.io/dataguide/types/relational/what-are-database-migrations#:~:text=Migrations%20help%20transition%20database%20schemas,structures%20in%20a%20programmatic%20way.).

This is the migration to create the relation,

```sql
CREATE TABLE user_profile (
  user_id integer PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  username VARCHAR(255) NOT NULL,
  password VARCHAR NOT NULL,
  --because we will use sha-512 hashing algorithm
  email VARCHAR NOT NULL,
  first_name VARCHAR,
  last_name VARCHAR
);
```

While this SQL reverts the database change

```sql
DROP TABLE user_profile
```

To execute the statements, use

```bash
diesel migration run
```

Should you have a reason to revert the state of the database run

```bash
diesel migration redo
```

Next the migration for the contacts relation. Generate a new migration directory `diesel migration generate create_contacts` for the relation and paste the sql statements for its creation and deletion.

```sql
CREATE TABLE contact (
  contact_id integer PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
  user_id integer REFERENCES user_profile(user_id),
  email VARCHAR NOT NULL,
  phone VARCHAR(30),
  city VARCHAR(255)
  country VARCHAR(255)
);
```

```sql
DROP TABLE contact
```

Finally, do the thing, `diesel migration run`.

| **NOTE:**                                                                           |
| :---------------------------------------------------------------------------------- |
| Trailing commas in SQL are not allowed. Also `user` is a reserved word in postgres. |

We can now cd into the `src` directory and write the code for integrating the db. Create `db.rs` which we will use to establish a connection to our database.

```bash
touch db.rs
```

db.rs code.

```rs
use diesel::pg::PgConnection;
use diesel::prelude::*;
use dotenvy::dotenv;
use std::env;

pub fn establish_db_connection() -> PgConnection {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    PgConnection::establish(&database_url).expect(&format!("Error connecting to {}", database_url))
}
```

Diesel cli also, generates a `schema.rs` file when we ran `diesel migration run`. It is a high level abstraction of our database, enabling us to safely interact with it.

Declare both files as a modules in `main.rs`.

```rs
mod db;
mod schema;
mod models;

fn main() {
    println!("Hello, world!");
}
```

Let us create our models directory, which will act as a representation of the individual entities in our db. In this directory, we will also create three new files.

1. `mod.rs` - enable us to share the code as a module to the main file.
2. `user.rs` - represent the user relation.
3. `contact.rs` - represent the contact relation.

```bash
mkdir models
cd models
touch mod.rs user.rs contact.rs
```

Declare the two modules in `mod.rs` as public to make them visible in main, i.e

```bash
pub mod contact;
pub mod user;
```

Lets write code for the user profile

```rs
use diesel::prelude::*;

#[derive(AsChangeset, Identifiable, Queryable, Selectable)]
#[diesel(primary_key(user_id))]
#[diesel(table_name = crate::schema::user_profile)]
#[diesel(check_for_backend(diesel::pg::Pg))]
pub struct UserProfile {
    user_id: i32,
    username: String,
    password: String,
    email: String,
    first_name: Option<String>,
    last_name: Option<String>,
}
```

Let's go over the attributes one by one,

1. `#[derive(AsChangeset, Identifiable, Queryable, Selectable)]`

    **Queryable** Will generate the code to load a UserProfile from a sql query. It assumes the order of your struct matches the columns of your relation.

    **Selectable** generates code to construct a matching `SELECT` clause based on the schema which we define using the `#[diesel(table_name = crate::schema::user_profile)]` attribute. In our `user_profile` model above, the raw sql statement would be `select * from user_profile`.

    **Identifiable** provides the necessary functionality to identify and retrieve records from a database. The raw sql query might be `select * from user_profile where id = 1`

    **AsChangeset** also enables use to update a record by passing a `&UserProfile` to `.set`. tl;dr - Provides the SET clause of the UPDATE statement.

2. `#[diesel(primary_key(user_id))]` - We use this attribute to tell diesel what our primary key is.

3. `#[diesel(table_name = crate::schema::user_profile)]` - as explained above is what makes the **Selectable** trait work.

4. #[diesel(check_for_backend(diesel::pg::Pg))] add compile time checks for our types, improving on the error message we receive.

Now for the `contact` model

```rs
use diesel::prelude::*;

use crate::models::user::UserProfile;

#[derive(AsChangeset, Identifiable, Queryable, Selectable, Associations)]
#[diesel(belongs_to(UserProfile, foreign_key = user_id))]
#[diesel(primary_key(contact_id))]
#[diesel(table_name = crate::schema::contact)]
#[diesel(check_for_backend(diesel::pg::Pg))]
pub struct Contact {
    contact_id: i32,
    user_id: Option<i32>,
    email: String,
    phone: Option<String>,
    city: Option<String>,
    country: Option<String>,
}
```

Two important additions are to be noted here.

1. `#[derive(AsChangeset, Identifiable, Queryable, Selectable, Associations)]`. The use of **Associations** trait, which enables us represent a one-many relationship between our `user_profile` and `contact` relation.

2. `#[diesel(belongs_to(UserProfile, foreign_key = user_id))]` - Here, we tell diesel the name of our foreign key.

### Implementing our Crud Operations <a name="crud-impl"></a>

We will get started by creating a `data` directory in the `src` directory, and in it create the `user_repository` and `contact_repository` files.

The two files will handle all our database logic for **reads**, **writes**, **updates** and **deletions** of records in our database.

```bash
mkdir data
cd data
touch mod.rs user_repository.rs contact_repository.rs
```

Update main.rs to include our module.

```rs
mod db;
mod data;
mod models;
mod schema;

...
```

Update the `src/data/mod.rs` to recognize our `user_repository.rs` and `contact_repository.rs` mods.

```rs
mod contact_repository;
mod user_repository;
```

We will first work on inserting a user to the users relation. Let's head over back to our users model - `src/models/users.rs`

We have two approaches to inserting data using diesel.

The first is by using tuples. The following Rust, diesel code

```rs
use crate::schema::user_profile::dsl::*;
use diesel::*;

let conn = &mut establish_db_connection();

insert_into(user_profile)
    .values((
        username.eq("shadow"),
        password.eq("176c1e..50c7e"),
        email.eq("cidkage@shadowgarden.com"),
        first_name.eq("Cid"),
        last_name.eq("Kagenou"),
    ))
    .execute(conn)
    .unwrap();
```

Would be equivalent to the following SQL statement.

```sql
INSERT INTO user_profile (username, password, email, first_name, last_name)
VALUES ('shadow', '176c1e..50c7e', 'cidkage@shadowgarden.com', 'Cid', 'Kagenou');
```

This would still work, but it's cumbersome, especially if we had large relations and had to deserialize data everytime we received it.

The second approach is using the `Insertable` trait. Once derived, it allows us to map our relations to a struct defined in our code. This is the approach we will use.

In our case, let us create a new struct `NewUserProfile` in the `users.rs` module, that will be responsible to the creation of new users.

```rs
#[derive(Insertable)]
#[diesel(table_name = crate::schema::user_profile)]
pub struct NewUserProfile {
    pub username: String,
    pub password: String,
    pub email: String,
    pub first_name: Option<String>,
    pub last_name: Option<String>,
}
```

You might be wondering why we haven't included `user_id` in our new struct, and that is because of this statement `user_id integer PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,` we wrote when creating our relations. It autoincrements the value of `user_id` each time a new record is inserted.

We are now ready to write code to handle our **CRUD** operations.

Populate your `src/data/user_repository` with the following,

```rs
use crate::models::user::{NewUserProfile, UserProfile};
use crate::schema::user_profile;
use diesel::pg::PgConnection;
use diesel::prelude::*;

pub fn create_user(conn: &mut PgConnection, new_user: NewUserProfile) -> UserProfile {
    diesel::insert_into(user_profile::table)
        .values(&new_user)
        .returning(UserProfile::as_returning())
        .get_result(conn)
        .expect("error inserting new record into database")
}
```

We have the `create_user` function that takes a `conn` parameter to our database connection and `new_user` which will contain the new values we want to insert into our database.

Some points to note about the functions:

-   Our database supports the [`RETURNING`](https://www.postgresql.org/docs/current/dml-returning.html) clause and that is why we are able to use the `get_result` method.
-   This clause is what allows us to get back rows of all the inserted rows, i.e `-> UserProfile`.

    We could probably do better by returning a results but we'll save that for next time.

We can do the same for our contacts. Edit your `src/models/contact.rs` to include the data model for a new contact in initialized with the`NewContact` struct.

```rs
#[derive(Insertable, Debug)]
#[diesel(table_name = crate::schema::contact)]
pub struct NewContact {
    user_id: Option<i32>,
    email: String,
    phone: Option<String>,
    city: Option<String>,
    country: Option<String>,
}
```

And finally, populate your `src/data/contact_repository.rs` with the function to create a new contact record in our contacts relation.

```rs
use crate::models::contact::{Contact, NewContact};
use crate::schema::contact;
use diesel::pg::PgConnection;
use diesel::prelude::*;

pub fn create_contact(conn: &mut PgConnection, new_contact: NewContact) -> Contact {
    diesel::insert_into(contact::table)
        .values(&new_contact)
        .returning(Contact::as_returning())
        .get_result(conn)
        .expect("error inserting new contact record into database")
}
```

We are ready to write test for our methods, but before moving forward, let's format our code and run the Rust linter.

```bash
cargo fmt --all
cargo clippy
```

As show in the screenshot below, `cargo clippy` generates a list of warning and even suggests changes we can make to improve our codebase.

![Screenshot of warnings generated by clippy](/contact-backend/cargo-clippy-warning-1.png "clippy warnings")

We can ignore the first two about `dead_code` as we have yet to make us of our `create_user` and `create_contact` functions yet.

Let us tackle the third warning. The warning here is that `expect` will [still execute](https://rust-lang.github.io/rust-clippy/master/index.html#expect_fun_call) the next line, not matter the `Result` we obtain in this line. To fix the warning, clippy suggests we use the [`unwrap_or_else`](https://doc.rust-lang.org/core/result/enum.Result.html#method.unwrap_or_else) method from `Result`, which takes a closure that will be executed should our statement return the `Err` variant of `Result`.

Since, there isn't much we can do without a database url, the right call IMO should be to `panic!`. Lets head over to the `src/db.rs` file and change our `establish_db_connection` function to,

```rs
pub fn establish_db_connection() -> PgConnection {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL").unwrap_or_else(|err| {
        panic!("check your DATABASE_URL env configuration: {err}");
    });
    PgConnection::establish(&database_url).unwrap_or_else(|err| {
        panic!("Error connecting to the database: {err}");
    })
}
```

Should you be writing production code, you should be handling these `Err`s more gracefully. Diesel provides a non-exhaustive [`ConnectionError`](https://docs.diesel.rs/master/diesel/result/enum.ConnectionError.html) enum that you should utilize.

### Tests for our database <a name="db-tests"></a>

Let's write some simple unit tests, to test the validity of our diesel code. We will later use this for our actix tests too.

```bash
mkdir tests
cd tests
touch mod.rs db_tests.rs
```

Declare the tests module in `main.rs`.

```rs
mod db;
mod data;
mod models;
mod schema;
mod tests;

...
```

And in `src/tests/mod.rs`, declare the `db_tests.rs` module

```rs
mod db_tests;
```

We will use the [`test_transaction`](https://docs.rs/diesel/latest/diesel/prelude/trait.Connection.html#method.test_transaction) method which starts a transaction and is rolled back at the end of the test. We will test **reads**, **writes** to the `user_profile` relation.

Populate your test code like this

```rs
#[cfg(test)]
mod tests {
    use crate::data::user_repository;
    use crate::db::establish_db_connection;
    use crate::models::user::{NewUserProfile, UserProfile};
    use crate::schema::user_profile::dsl::*;
    use diesel::prelude::*;
    use diesel::result::Error;

    // test writing and reading to the user_profile relation
    fn read_user_profile_records(conn: &mut PgConnection) -> QueryResult<Vec<UserProfile>> {
        user_profile
            .select(UserProfile::as_select())
            .load::<UserProfile>(conn)
    }

    #[test]
    fn test_user_profile_write() {
        let connection = &mut establish_db_connection();

        let new_user = NewUserProfile {
            username: "shadow".to_string(),
            password: "12312".to_string(),
            email: "fake@email.com".to_string(),
            first_name: Some("shadow".to_string()),
            last_name: None,
        };

        connection.test_transaction::<_, Error, _>(|conn| {
            let created_user = user_repository::create_user(conn, new_user.clone());

            assert_eq!(created_user.email, new_user.email);
            println!("user id {:?}", created_user.user_id);

            let res = read_user_profile_records(conn).unwrap();
            assert_eq!(res.len(), 1);

            Ok(())
        });
    }
}
```

You might be asking why we are using a mutable reference for the connection variable, and the answer to that would be because [diesel 2.0](https://diesel.rs/guides/migration_guide.html#2-0-0-mutable-connection) changed their API.

We can call it a day and finish testing **updates** and **deletions**. For that remember we derived the [`AsChangeSet`](https://docs.diesel.rs/2.1.x/diesel/prelude/derive.AsChangeset.html) trait on our `UserProfile` struct in `src/models/user.rs`

As before, we will need to first create a function to update our relation row in the `src/data/contact_repository.rs`.

```rs
pub fn update_user_profile_by_id(
    conn: &mut PgConnection,
    updated_user_profile: UserProfile,
) -> UserProfile {
    diesel::update(user_profile::table)
        .set(updated_user_profile)
        .get_result(conn)
        .expect("error updating specified recor")
}
```

Passing in our `UserProfile` itself is how we are selecting the user_profile to update. It is equivalent to doing, `update(user_profile.find(user_profile.user_id))` or `update(user_profile.filter(user_id.eq(user_profile.user_id)))`

In our `src/db_tests.rs` file, we are going to create a function that abstracts the creation of a new user for us and use it instead of declaring a `new_user` with struct literals in each test.

```rs
//generates a dummy user for us
fn generate_dummy_user_profile() -> NewUserProfile {
    NewUserProfile {
        username: "shadow".to_string(),
        password: "12312".to_string(),
        email: "fake@email.com".to_string(),
        first_name: Some("shadow".to_string()),
        last_name: None,
    }
}
```

We want to update the `username`, `first_name` and `last_name` fields. Like JavaScript rust also has the **spread operator** which we will make use of for the update test.

```rs
#[test]
fn test_user_profile_update_and_delete() {
    let connection = &mut establish_db_connection();

    let new_user = generate_dummy_user_profile();

    connection.test_transaction::<_, Error, _>(|conn| {
        let created_user = user_repository::create_user(conn, new_user.clone());

        let _username = String::from("sasuga_shadow_sama");
        let _first_name = Some(String::from("Cid"));
        let _last_name = Some(String::from("Kagenou"));

        let updated_user = UserProfile {
            username: _username.clone(),
            first_name: _first_name.clone(),
            last_name: _last_name.clone(),
            ..created_user
        };

        let updated_user = user_repository::update_user_profile_by_id(conn, updated_user);

        //assert whether our values have been updated as expected
        assert_eq!(updated_user.username, _username);
        assert_eq!(updated_user.first_name, _first_name);
        assert_eq!(updated_user.last_name, _last_name);

        let res = read_user_profile_records(conn).unwrap();
        assert_eq!(res.len(), 1);

        Ok(())
    });
}
```

You might be wondering why the values that we will update have an underscore to them. Well, it is because diesel defined our fields for `UserProfile` as their own types in `src/schema.rs`. More on this in the [diesel guides](https://diesel.rs/guides/schema-in-depth.html).

The code up to this point can be found in [this commit](https://github.com/687c/contacts-backend/commit/6c6bae7ea698f9ac78cfdb6cd1f27b63520e4556).

In the [next part](../contacts-backend-part-2/) we will set up everything with actix and get our api ready.
