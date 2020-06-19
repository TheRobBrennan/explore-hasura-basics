This project was created to follow along with [Hasura Basics](https://hasura.io/learn/graphql/hasura/introduction/).

# Deploy Hasura

Click [https://heroku.com/deploy?template=https://github.com/hasura/graphql-engine-heroku](https://heroku.com/deploy?template=https://github.com/hasura/graphql-engine-heroku) for a one-click deployment to [Heroku](https://dashboard.heroku.com/apps).

All you have to do is:

- Specify an app name (e.g. `rb-learn-hasura-back-end`)
- Click `Deploy app`

To view your app, simply use your app name - `rb-learn-hasura-back-end` in my case - and go to [https://rb-learn-hasura-back-end.herokuapp.com/console](https://rb-learn-hasura-back-end.herokuapp.com/console) to view the Hasura console.

# Basic data modeling

We have two models in this app: `users` and `todos`, each with its own set of properties.

As we create tables using the console or directly on postgres, Hasura GraphQL engine creates GraphQL schema object types and corresponding query/mutation fields with resolvers automatically.

## Create table users

In the Hasura Console, head over to the `Data` tab section and click on `Create Table`.

The users table will have the following columns:

- `id` (type text),
- `name` (type text),
- `created_at` (type timestamp and default now())
- `last_seen` (type timestamp and nullable)

Be sure to choose `id` as the primary key.

Great! You have created the first table required for the app.

View your Hasura console - [https://rb-learn-hasura-back-end.herokuapp.com/console](https://rb-learn-hasura-back-end.herokuapp.com/console) - and click on the `GraphiQL` tab.

### Example - Insert a user using GraphQL mutations

Copy the following mutation into the window and press the `Play` button to execute it:

```gql
mutation {
  insert_users(objects: [{ id: "1", name: "Praveen" }]) {
    affected_rows
  }
}
```

### Example - Query the data that we just inserted

Copy the following query into the window and press the `Play` button to execute it:

```gql
query {
  users {
    id
    name
    created_at
  }
}
```

### Example - Subscribe to watch changes to the users table

Copy the following subscription query into the window and press the `Play` button to execute it:

```gql
subscription {
  users {
    id
    name
    created_at
  }
}
```

Notice how the `Play` button changes to a `Stop` button. This subscription is actively listening for changes.

Open the `Data` link in another tab. Manually add data to our database; then visit our current tab to see changes as they occur.

## Create table todos

In the Hasura Console, head over to the `Data` tab section and click on `Create Table`.

The todos table will have the following columns:

- id (type integer;auto-increment),
- title (type text),
- is_completed (type boolean and default false)
- is_public (type boolean and default false)
- created_at (type timestamp and default now())
- user_id (type text)

Be sure to choose `id` as the primary key.

### Example - Insert a todo using GraphQL mutations

Copy the following mutation into the window and press the `Play` button to execute it:

```gql
mutation {
  insert_todos(objects: [{ title: "My First Todo", user_id: "1" }]) {
    affected_rows
  }
}
```

### Example - Query the data that we just inserted

Copy the following query into the window and press the `Play` button to execute it:

```gql
query {
  todos {
    id
    title
    is_public
    is_completed
    user_id
  }
}
```

### Example - Subscribe to watch changes to the todos table

Copy the following subscription query into the window and press the `Play` button to execute it:

```gql
subscription {
  todos {
    id
    title
    is_public
    is_completed
    user_id
  }
}
```

Notice how the `Play` button changes to a `Stop` button. This subscription is actively listening for changes.

Open the `Data` link in another tab. Manually add data to our database; then visit our current tab to see changes as they occur.

# Relationships

Relationships enable you to make nested object queries if the tables/views in your database are connected.

GraphQL schema relationships can be either of

- object relationships (one-to-one)
- array relationships (one-to-many)

## Object relationships

Let's say you want to query todos and more information about the user who created it. This is achievable using nested queries if a relationship exists between the two. This is a one-to-one query and hence called an object relationship.

An example of such a nested query looks like this:

```gql
query {
  todos {
    id
    title
    user {
      id
      name
    }
  }
}
```

## Array relationships

Let's look at an example query for array relationships:

```gql
query {
  users {
    id
    name
    todos {
      id
      title
    }
  }
}
```

In this query, you are able to fetch users and for each user, you are fetching the todos (multiple) written by that user. Since a user can have multiple todos, this would be an array relationship.

Relationships can be captured by foreign key constraints. Foreign key constraints ensure that there are no dangling data. Hasura Console automatically suggests relationships based on these constraints.

Though the constraints are optional, it is recommended to enforce these constraints for data consistency.

The above queries won't work yet because we haven't defined the relationships yet. But this gives an idea of how nested queries work.

## Create Foreign Key

In the `todos` table, the value of `user_id` column must be ideally present in the `id` column of `users` table. Otherwise it would result in inconsistent data.

Postgres allows you to define foreign key constraint to enforce this condition.

Let's define one for the `user_id` column in `todos` table.

Head over to Console -> Data -> todos -> Modify page.

Scroll down to `Foreign Keys` section at the bottom and click on `Add`.

Select the `Reference table` as `users`

- Choose the `From` column as `user_id` and `To` column as `id`
- We are enforcing that the `user_id` column of `todos` table must be one of the values of `id` in `users` table.

Click on `Save` to create the foreign key.

## Create Relationship

Now that the foreign key constraint is created, Hasura Console automatically suggests relationships based on that.

Head over to `Relationships` tab under `todos` table and you should see a suggested object relationship.

Click on `Add` in the suggested object relationship.

Enter the relationship name as `user` (already pre-filled) and click on `Save`.

## Try out Relationship Queries

Let's explore the GraphQL APIs for the relationship created:

```gql
query {
  todos {
    id
    title
    user {
      id
      name
    }
  }
}
```

# Data Transformations

Postgres allows you to perform data transformations using:

- Views
- SQL Functions

In this example, we are going to make use of `Views`. This view is required by the app to find the users who have logged in and are online in the last 30 seconds.

## Create View

The SQL statement for creating this view looks like this:

```sql
CREATE OR REPLACE VIEW "public"."online_users" AS
 SELECT users.id,
    users.last_seen
   FROM users
  WHERE (users.last_seen >= (now() - '00:00:30'::interval));
```

Head to Console -> Data -> SQL page.

## Subscription to Online Users

Now let's test by making a subscription query to the `online_users` view:

```gql
subscription {
  online_users {
    id
    last_seen
  }
}
```

In another tab, update an existing user's `last_seen` value to see the subscription response getting updated. Enter the value as `now()` for the `last_seen` column and click on `Save`.

## Create relationship to user

Now that the view has been created, we need a way to be able to fetch user information based on the `id` column of the view. Let's create a manual relationship from the view `online_users` to the table users using the `id` column of the view.

Head to Console -> Data -> online_users -> Relationships page.

Add new relationship by choosing the relationship type to be `Object Relationship`. Enter the relationship name as `user`. Select the configuration for current column as `id` and the remote table would be `users` and the remote column would be `id` again.

Let's explore the GraphQL APIs for the relationship created:

```gql
query {
  online_users {
    id
    last_seen
    user {
      id
      name
    }
  }
}
```

# Authorization

In this part of the tutorial, we are going to define role based access control rules for each of the models that we created.

Access control rules help in restricting querying on a table based on certain conditions.

In this realtime todo app use-case, we need to restrict all querying only for logged in users. Also certain columns in tables do not need to be exposed to the user.

The aim of the app is to allow users to manage their own todos only but should be able to view all the public todos.

We will define all of these based on role based access control rules in the subsequent steps.

## Setup todos table permissions

Head over to the Permissions tab under todos table to add relevant permissions.

### Insert permission

- In the enter new role textbox, type in “user”
- Click on edit (pencil) icon for “insert” permissions. This would open up a section below which lets you configure custom checks and allow columns.
- In the custom check, choose the following condition - `{"user_id":{"_eq":"X-Hasura-User-Id"}}`

Now under column insert permissions, select the `title` and `is_public` columns.

Finally under column presets, select `user_id` from `from session variable` mapping to `X-HASURA-USER-ID`.

Note: Session variables are key-value pairs returned from the authentication service for each request. When a user makes a request, the session token maps to a `USER-ID`. This `USER-ID` can be used in a permission to show that inserts into a table are only allowed if the `user_id` column has a value equal to that of `USER-ID`, the session variable.

Click on `Save Permissions`.

### Select permission

Now click on edit icon for "select" permissions. In the custom check, choose the following condition - `{"_or":[{"is_public":{"_eq":true}},{"user_id":{"_eq":"X-Hasura-User-Id"}}]}`

Under column select permissions, select all the columns.

Click on `Save Permissions`

### Update permission

Now click on edit icon for "update" permissions. In the custom check, choose `With same custom checks as insert`.

And under column update permissions, select the `is_completed` column.

Click on `Save Permissions` once done.

### Delete permission

Finally for delete permission, under custom check, choose `With same custom checks as insert, update`.

Click on `Save Permissions` and you are done with access control for `todos` table.

## Setup users table permissions

We also need to allow select and update operations into `users` table. On the left sidebar, click on the `users` table to navigate to the users table page and switch to the `Permissions` tab.

### Select permission

Click on the Edit icon (pencil icon) to modify the select permission for role user. This would open up a section below which lets you configure its permissions.

Here the users should be able to access every other user's `id` and `name` data - `without any checks` in `Row select permission`.

Click on `Save Permissions`

### Update permission

The user who is logged in should be able to modify only his own record. So let’s set that permission now.

In the Row update permission, under custom check, choose the following condition - `{"id":{"_eq":"X-Hasura-User-Id"}}`

Under column update permissions, select `last_seen` column, as this will be updated from the frontend app.

Click on `Save Permissions` and you are done with access control rules for `users` table.

## Setup online_users view permissions

Head over to the Permissions tab under `online_users` view to add relevant permissions.

### Select permission

Here in this view, we only want the user to be able to select data and not do any mutations. Hence we don't define any permission for insert, update or delete.

For Row select permission, choose `Without any checks` and under Column select permission, choose both the columns `id` and `last_seen`.

Click on `Save Permissions`. You have completed all access control rules required for the realtime todo app.

# Authentication

In this part, we will look at how to integrate an Authentication provider.

The realtime todo app needs to be protected by a login interface. We are going to use [Auth0](https://auth0.com/) as the identity/authentication provider for this example.

Note: [Auth0](https://auth0.com/) has a free plan for up to 7000 active users.

The basic idea is that, whenever a user authenticates with [Auth0](https://auth0.com/), the client app receives a token which can be sent in the `Authorization` headers of all GraphQL requests. Hasura GraphQL Engine would verify if the token is valid and allow the user to perform appropriate queries.

Let's get started!

## Create Auth0 App

## Rules for Custom JWT Claims

## Connect Hasura with Auth0

## Sync Users with Rules

## Test with Auth0 Token

# Custom Business Logic

## Creating Actions

## Write custom resolvers

## Add Remote Schema

## Write event webhook

## Create event trigger

# What's next?
