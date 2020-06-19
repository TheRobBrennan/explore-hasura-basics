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
