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

- Navigate to the [Auth0 Dashboard](https://manage.auth0.com/)
- Signup / Login to the account
- Create a new tenant.
- Click on the `Applications` menu option on the left and then click the `+ Create Application` button.
- In the Create Application window, set a name for your application and select `Single Page Web Applications`. (Assuming the frontend app will be an SPA built on react/vue etc)
- In the `settings` of the application, we will add appropriate (e.g: http://localhost:3000/callback) URLs as `Allowed Callback URLs` and `Allowed Web Origins`. We can also add domain specific URLs as well for the app to work. (e.g: https://myapp.com/callback).

This would be the URL of the frontend app which you will deploy later. You can ignore this, for now. You can always come back later and add the necessary URLs.

## Rules for Custom JWT Claims

[Custom claims](https://auth0.com/docs/scopes/current/custom-claims) inside the JWT are used to tell Hasura about the role of the caller, so that Hasura may enforce the necessary authorization rules to decide what the caller can and cannot do. In the Auth0 dashboard, navigate to [Rules](https://manage.auth0.com/#/rules).

Add the following rule to add our custom JWT claims under `hasura-jwt-claim`:

```js
function (user, context, callback) {
  const namespace = "https://hasura.io/jwt/claims";
  context.idToken[namespace] =
    {
      'x-hasura-default-role': 'user',
      // do some custom logic to decide allowed roles
      'x-hasura-allowed-roles': ['user'],
      'x-hasura-user-id': user.user_id
    };
  callback(null, user, context);
}
```

## Connect Hasura with Auth0

In this part, you will learn how to connect Hasura with the Auth0 application that you just created in the previous step.

We need to configure Hasura to use the Auth0 public keys. An easier way to generate the config for JWT is:

- Click on the following link - [https://hasura.io/jwt-config](https://hasura.io/jwt-config)
- For `Select Provider` choose `Auth0`
- Enter `Auth0 Domain Name` (e.g. `demo-explore-hasura-basics.us.auth0.com`)
- Click `Generate Config`

The generated configuration can be used as the value for environment variable `HASURA_GRAPHQL_JWT_SECRET`.

Since we have deployed Hasura GraphQL Engine on Heroku, let's head to Heroku dashboard to configure the admin secret and JWT secret.

Open the "Settings" page for your Heroku app, add a new Config Var called `HASURA_GRAPHQL_JWT_SECRET`, and copy and paste the generate JWT configuration into the value box.

Next, create a new Config Var called `HASURA_GRAPHQL_ADMIN_SECRET` and enter a secret key to protect the GraphQL endpoint. (Imagine this as the password to your GraphQL server).

Great! Now your Hasura GraphQL Engine is secured using Auth0.

## Sync Users with Rules

Auth0 has rules that can be set up to be called on every login request. We need to set up a rule in Auth0 which allows the users of Auth0 to be in sync with the users in our database. The following code snippet allows us to do the same. Again using the Rules feature, create a new blank rule `upsert-user` and paste in the following code snippet:

```js
function (user, context, callback) {
  const userId = user.user_id;
  const nickname = user.nickname;

  // Modify with your Hasura admin secret and URL to the application
  const admin_secret = "demo";
  const url = "https://rb-learn-hasura-back-end.herokuapp.com/v1/graphql";

  request.post({
      headers: {'content-type' : 'application/json', 'x-hasura-admin-secret': admin_secret},
      url:   url,
      body:    `{\"query\":\"mutation($userId: String!, $nickname: String) {\\n          insert_users(\\n            objects: [{ id: $userId, name: $nickname }]\\n            on_conflict: {\\n              constraint: users_pkey\\n              update_columns: [last_seen, name]\\n            }\\n          ) {\\n            affected_rows\\n          }\\n        }\",\"variables\":{\"userId\":\"${userId}\",\"nickname\":\"${nickname}\"}}`
  }, function(error, response, body){
       console.log(body);
       callback(null, user, context);
  });
}
```

Note: Modify `x-hasura-admin-secret` and `url` parameters appropriately according to your app. Here we are making a simple request to make a mutation into users table.

That’s it! This rule will now be triggered on every successful signup or login, and we insert or update the user data into our database using a Hasura GraphQL mutation.

The above request performs a mutation on the users table with the id and name values.

## Test with Auth0 Token

Hasura is configured to be used with Auth0. Now let's test this setup by getting the token from Auth0 and making GraphQL queries with the Authorization headers to see if the permissions are applied.

To get a JWT token for testing,

Copy this URL - https://auth0-domain.auth0.com/login?client=client_id&protocol=oauth2&response_type=token%20id_token&redirect_uri=callback_uri&scope=openid%20profile and update the URL as given below:

- Replace auth0-domain with the one we created in the previous steps.
- Replace client_id with Auth0 application's client_id.
- Replace callback_uri with http://localhost:3000/callback for testing. You don't need anything to run on localhost:3000 for this to work.
- Make sure http://localhost:3000/callback has been added under Allowed Callback URLs in the Auth0 app settings.

For my demo project, the URL will be updated to be https://demo-explore-hasura-basics.us.auth0.com/login?client=8fCQESPB57A48QvT0LM5edcaxbq7JrhE&protocol=oauth2&response_type=token%20id_token&redirect_uri=http://localhost:3000/callback&scope=openid%20profile

Now try entering the updated URL in the browser. It should take you to the Auth0 login screen.

After successfully logging in, you will be redirected to something like:

http://localhost:3000/callback#access_token=CkxPdM_gMZz3ghILpVyRd4qCoJ_T1hIZ&scope=openid%20profile&expires_in=7200&token_type=Bearer&state=eBssl0ntJvATx98ha07BhZsBzcAkRSdp&id_token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Inh0Vl9kbE1HZ0VydVZxT2RjV3NFVSJ9.eyJodHRwczovL2hhc3VyYS5pby9qd3QvY2xhaW1zIjp7IngtaGFzdXJhLWRlZmF1bHQtcm9sZSI6InVzZXIiLCJ4LWhhc3VyYS1hbGxvd2VkLXJvbGVzIjpbInVzZXIiXSwieC1oYXN1cmEtdXNlci1pZCI6Imdvb2dsZS1vYXV0aDJ8MTE2MDU4NjY4MzAyMjkwODYxODEwIn0sImdpdmVuX25hbWUiOiJSb2IiLCJmYW1pbHlfbmFtZSI6IkJyZW5uYW4iLCJuaWNrbmFtZSI6InJvYiIsIm5hbWUiOiJSb2IgQnJlbm5hbiIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS0vQU9oMTRHakUwdUR2eTFMRWlVS3FQTWVvTzdjYXVWckJRMXl0VjQ3aUpGUWVvT1EiLCJnZW5kZXIiOiJtYWxlIiwibG9jYWxlIjoiZW4iLCJ1cGRhdGVkX2F0IjoiMjAyMC0wNi0xOVQyMzoyOTowOC42NjVaIiwiaXNzIjoiaHR0cHM6Ly9kZW1vLWV4cGxvcmUtaGFzdXJhLWJhc2ljcy51cy5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMTYwNTg2NjgzMDIyOTA4NjE4MTAiLCJhdWQiOiI4ZkNRRVNQQjU3QTQ4UXZUMExNNWVkY2F4YnE3SnJoRSIsImlhdCI6MTU5MjYwOTM2MywiZXhwIjoxNTkyNjQ1MzYzLCJhdF9oYXNoIjoidWJwX0c4UUJqRmNXazkwaW9DZms0QSIsIm5vbmNlIjoibnpvQXNKYmhMd3ZsMF82dEpKWXBod0M5Y0JKTU4tWFEifQ.BhR2FViuZXufd7euZuuWsB8gKlJqNRAPckEWLh3a0biQZ7X57a3op4fQ5dKJtceBWhGFZRMeHBFtPue6ZZ--WZYKnW2pC7Rzr3A0jXa1pYFiXaDWJ6tTOzXpY2NWS3CjIGZzCeJhbmlhrRhkuSayBkylK2rHtmPcgal_dGbQng1vYGJKXi-qu89EzkEJgrebMaiBolnYBGI40orP4qjMYhrfrNLdG8X8RUDDSJGCQpeYWWAH8hvSV8SDAO5s3JkA2LgV7P1pF24X8FMeeejhO7z1jAO_vfV0Rog8r7v4PcwXCY0B5p7xEQqdSkseWwtCZriAxpTjuDZ76ExsmTAjgw

This page will be a 404, unless you are running some other server on that port locally. We care only about the URL parameters.

Extract the `id_token` value from this URL. This is the JWT:

```js
// Paste the JWT into the jwt.io debugger
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Inh0Vl9kbE1HZ0VydVZxT2RjV3NFVSJ9.eyJodHRwczovL2hhc3VyYS5pby9qd3QvY2xhaW1zIjp7IngtaGFzdXJhLWRlZmF1bHQtcm9sZSI6InVzZXIiLCJ4LWhhc3VyYS1hbGxvd2VkLXJvbGVzIjpbInVzZXIiXSwieC1oYXN1cmEtdXNlci1pZCI6Imdvb2dsZS1vYXV0aDJ8MTE2MDU4NjY4MzAyMjkwODYxODEwIn0sImdpdmVuX25hbWUiOiJSb2IiLCJmYW1pbHlfbmFtZSI6IkJyZW5uYW4iLCJuaWNrbmFtZSI6InJvYiIsIm5hbWUiOiJSb2IgQnJlbm5hbiIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS0vQU9oMTRHakUwdUR2eTFMRWlVS3FQTWVvTzdjYXVWckJRMXl0VjQ3aUpGUWVvT1EiLCJnZW5kZXIiOiJtYWxlIiwibG9jYWxlIjoiZW4iLCJ1cGRhdGVkX2F0IjoiMjAyMC0wNi0xOVQyMzoyOTowOC42NjVaIiwiaXNzIjoiaHR0cHM6Ly9kZW1vLWV4cGxvcmUtaGFzdXJhLWJhc2ljcy51cy5hdXRoMC5jb20vIiwic3ViIjoiZ29vZ2xlLW9hdXRoMnwxMTYwNTg2NjgzMDIyOTA4NjE4MTAiLCJhdWQiOiI4ZkNRRVNQQjU3QTQ4UXZUMExNNWVkY2F4YnE3SnJoRSIsImlhdCI6MTU5MjYwOTM2MywiZXhwIjoxNTkyNjQ1MzYzLCJhdF9oYXNoIjoidWJwX0c4UUJqRmNXazkwaW9DZms0QSIsIm5vbmNlIjoibnpvQXNKYmhMd3ZsMF82dEpKWXBod0M5Y0JKTU4tWFEifQ.BhR2FViuZXufd7euZuuWsB8gKlJqNRAPckEWLh3a0biQZ7X57a3op4fQ5dKJtceBWhGFZRMeHBFtPue6ZZ--WZYKnW2pC7Rzr3A0jXa1pYFiXaDWJ6tTOzXpY2NWS3CjIGZzCeJhbmlhrRhkuSayBkylK2rHtmPcgal_dGbQng1vYGJKXi-qu89EzkEJgrebMaiBolnYBGI40orP4qjMYhrfrNLdG8X8RUDDSJGCQpeYWWAH8hvSV8SDAO5s3JkA2LgV7P1pF24X8FMeeejhO7z1jAO_vfV0Rog8r7v4PcwXCY0B5p7xEQqdSkseWwtCZriAxpTjuDZ76ExsmTAjgw
```

Test this JWT in [jwt.io](https://jwt.io/) debugger.

The debugger should give you the decoded payload that contains the JWT claims that have been configured for Hasura under the key `https://hasura.io/jwt/claims`. Inside this object, the role information will be available under `x-hasura-default-role` and `x-hasura-allowed-roles` keys; user-id information will be available under `x-hasura-user-id` key.

```js
// Decoded - Header (Algorithm and Token Type)
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "xtV_dlMGgEruVqOdcWsEU"
}

// Decoded - Payload
{
  "https://hasura.io/jwt/claims": {
    "x-hasura-default-role": "user",
    "x-hasura-allowed-roles": [
      "user"
    ],
    "x-hasura-user-id": "google-oauth2|116058668302290861810"
  },
  "given_name": "Rob",
  "family_name": "Brennan",
  "nickname": "rob",
  "name": "Rob Brennan",
  "picture": "https://lh3.googleusercontent.com/a-/AOh14GjE0uDvy1LEiUKqPMeoO7cauVrBQ1ytV47iJFQeoOQ",
  "gender": "male",
  "locale": "en",
  "updated_at": "2020-06-19T23:29:08.665Z",
  "iss": "https://demo-explore-hasura-basics.us.auth0.com/",
  "sub": "google-oauth2|116058668302290861810",
  "aud": "8fCQESPB57A48QvT0LM5edcaxbq7JrhE",
  "iat": 1592609363,
  "exp": 1592645363,
  "at_hash": "ubp_G8QBjFcWk90ioCfk4A",
  "nonce": "nzoAsJbhLwvl0_6tJJYphwC9cBJMN-XQ"
}
```

# Custom Business Logic

Hasura gives you CRUD + realtime GraphQL APIs with authorization & access control. However, there are cases where you would want to add custom/business logic in your app. For example, in the todo app that we are building, before inserting todos into the public feed we want to validate the text for profanity.

Custom business logic can be handled in a few flexible ways in Hasura:

- Actions
- Remote Schemas
- Event Triggers

Actions are a way to extend Hasura’s schema with custom business logic using custom queries and mutations. Actions can be added to Hasura to handle various use cases such as `data validation`, `data enrichment from external sources` and any other `complex business logic`.

![https://hasura.io/docs/1.0/_images/actions-arch1.png](https://hasura.io/docs/1.0/_images/actions-arch1.png)

Actions can be either a Query or a Mutation.

- `Query Action` - If you are querying some `data from an external API` or doing some `validations / transformations` before sending back the data, you can use a Query Action.
- `Mutation Action` - If you want to perform data validations or do some custom logic before manipulating the database, you can use a Mutation Action.

Remote schemas - Hasura has the ability to merge remote GraphQL schemas and provide a unified GraphQL API. Think of it like automated schema stitching. This way, we can write custom GraphQL resolvers and add it as a remote schema.

![https://hasura.io/docs/1.0/_images/remote-schemas-arch1.png](https://hasura.io/docs/1.0/_images/remote-schemas-arch1.png)

If you are choosing between `Actions` and `Remote Schemas`, here's something to keep in mind:

- Use `Remote Schemas` if you have a GraphQL server or if you're comfortable building one yourself.
- Use `Actions` if you need to call a REST API

Event Triggers - Hasura can be used to create event triggers on tables in the Postgres database. Event triggers reliably capture events on specified tables and invoke webhooks to carry out any custom logic. After a mutation operation, triggers can call a webhook asynchronously.

**Use case for the todo app**

In the todo app backend that you have built, there are certain custom functionalities you may want to add:

If you want to fetch profile information from Auth0, you need to make an API call to Auth0 with the token. Auth0 only exposes a REST API and not GraphQL. This API has to be exposed to the GraphQL client.

We will add an Action in Hasura to extend the API. We will also see how the same thing can be done with a custom GraphQL server added as a Remote Schema.

Get notified via email whenever a new user registers in your app. This is an asynchronous operation that can be invoked via Event trigger webhook.

We will see how these 2 use-cases can be handled in Hasura.

## Creating Actions

## Write custom resolvers

## Add Remote Schema

## Write event webhook

## Create event trigger

# What's next?
