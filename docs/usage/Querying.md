# Querying

Models can be used inside of **query tools** to generate a GraphQL query and send it off to the server.

#### `query` and `mutate`

These two query tools are used for making mutations or queries and are used identically to each other.

Lets use them to query for some users and their pets

```javascript

...

const {models, query, mutate, fetch} = new Modelizr({
  models: { ... },
  config: { ... }
})

const {User, Dog, Cat, Animal} = models

query(
	User(
		Animal(
			Dog, Cat
		)
	)
).then((res, normalize) => {
	
})
```

Internally this will generate the following query and post it to `http://path.to.api/graphql` - as defined with the `.path()` modifier.

```
{
  users (ids: [1, 2, 3]) {
     id,
     firstname,
     lastname
     books {
        id,
        title,
        publisher
     }
  }
}
```

We can make a similar query for books and their respective authors, although we will need to use an `as(key)` modifier to alter the models key.

```javascript
query(
    book(
        user().as("author")
    ).params({ids: [1, 2, 3]})
)
    .path('http://path.to.api/graphql')
    .then((res, normalize) => {
        // res -> the response from the server
        // normalize(res.body) // normalized response
    })
```
An alternative to using `.as()` is to create an alias of the user model
```javascript
import { alias } from 'modelizr'

const author = alias(user, "author")

query(
    book(
        author()
    )
) ...
```
The resulting query will look like this:
```
{
  books (ids: [1, 2, 3]) {
     id,
     title,
     publisher
     author {
        id,
        firstName,
        lastName
     }
  }
}
```

You can also use the `.normalize()` modifier instead of `.then()` to directly normalize the servers response.

```javascript
query( ... ).normalize(res => {
    // res -> normalized response
})
```

When using unions in a query, modelizr will prefix child keys with `... on`. For instance the following query:

```javascript
query(
    owner(
        user(),
        group()
    )
)
```

Will generate
```
{
  owners {
     id,
     ... on users {
        id,
        firstName,
        lastName
     },
     ... on groups {
        id,
        users {
            id,
            firstName,
            lastName
        }
     }
  }
}
```

#### `mutation`

To make GraphQL mutations, we can use the `mutation()` tool. This works similarly to the `query()` tool although with a slightly different query generator. Lets mutate a **user**.

The addition of the `.query()` modifier is to declare that we want the mutated user to be returned by the GraphQL server. This may become the default in future, but for
now you will need to explicitly specify this.

```javascript
import { mutation } from 'graphql'

mutation
    .as("createUser")
    .params({admin: true})
    .path('http://path.to.api/graphql')
    .query()

mutation(
    user({firstName: "John", lastName: "Doe"})
).normalize(res => {})
```
**Note** - We passed params directly into the model without the use of a modifier. This makes more sense when making single-model mutations.

This will create a query that looks as follows.
```
mutation createUser(admin: true) {
 users(firstName: "John", lastName: "Doe") {
   id,
   firstName,
   lastName
 }
}
```

#### `request`

Finally we have the `request()` tool. This is for making a request to a non GraphQL server, where the returned data can still be expected to match our defined models.
You will need to explicitly give this request a body.

```javascript
import { request } from 'modelizr'

request
    .body({
        firstName: "John",
        lastName: "Doe"
    })
    .path("http://...")
    .method("POST")
    .contentType( ... )
    .headers( ... )

request(
    user(
        book()
    )
).then((res, normalize) => {})
```