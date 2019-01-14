
<h1 align="center">💂‍♀️ GraphQL Rate Limit 💂‍♂️</h1>

<p align="center">
A GraphQL directive to add basic but granular rate limiting to your Queries or Mutations.
</p>

---

## Features

- 💂‍♀️ Add rate limits to queries or mutations 
- 🔑 Add filters to rate limits based on the query or mutation args
- ❌ Custom error messaging
- ⏰ Configure using a simple `max` per `window` arguments
- 💼 Custom stores, use Redis, Postgres, Mongo... it defaults to in-memory
- 💪 Written in TypeScript


## Install

```sh
yarn add graphql-rate-limit
```

### Example

```graphql
directive @rateLimit(
  max: Int, 
  window: String,
  message: String, 
  identityArgs: [String], 
) on FIELD_DEFINITION

type Query {
  # Rate limit to 5 per second
  getItems: [Item] @rateLimit(window: "1s", max: 5)

  # Rate limit access per item ID
  getItem(id: ID!): Item @rateLimit(identityArgs: ["id"])
}

type Mutation {
  # Rate limit with a custom error message
  createItem(title: String!): Item @rateLimit(message: "You are doing that too often.")
}
```

### Usage

##### Step 1. 

Create a configured GraphQLRateLimit class.

```js
const { createRateLimitDirective } = require('graphql-rate-limit');
// OR
import { createRateLimitDirective } from 'graphql-rate-limit';

const GraphQLRateLimit = createRateLimitDirective({
  /**
   * `identifyContext` is required and used to identify the user/client. The most likely cases
   * are either using the context's request.ip, or the user ID on the context.
   * A function that accepts the context and returns a string that is used to identify the user.
   */
  identifyContext: (ctx) => ctx.user.id, // Or could be something like: return ctx.req.ip;
  /**
   * `store` is optional as it defaults to an InMemoryStore. See the implementation of InMemoryStore if 
   * you'd like to implement your own with your own database.
   */
  store: new MyCustomStore(),
  /**
   * Generate a custom error message. Note that the `message` passed in to the directive will be used 
   * if its set.
   */
  formatError: ({ fieldName }) => {
    return `Woah there, you are doing way too much ${fieldName}`
  }
});
```

#### Step 2.

Add GraphQLRateLimit to your GraphQL server configuration. Example using Apollo Server:

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  schemaDirectives: {
    rateLimit: GraphQLRateLimit
  }
});
```

#### Step 3.

Use in your GraphQL Schema.

```graphql
# This must be added to the top of your schema.
directive @rateLimit(
  max: Int, 
  window: String,
  message: String, 
  identityArgs: [String], 
) on FIELD_DEFINITION

type Query {
  # Limit queries to getThings to 10 per minute.
  getThings: [Thing] @rateLimit(max: 10, window: "6s")
}

type Query {
  # Limit attempts to login with a particular email to 10 per 2 hours.
  login(email: String!, password: String!): String @rateLimit(max: 10, window: "2h", identityArgs: ["email"])
}
```

**Note:** We use Zeit's `ms` to parse the `window` arg, [docs here](https://github.com/zeit/ms).

### Redis Store usage

It is recommended to use a persistent store rather than the default InMemoryStore. 
GraphQLRateLimit supports Redis. You need to install Redis in your project first. 

```js
import * as graphqlRateLimit from 'graphql-rate-limit';

const GraphQLRateLimit = graphqlRateLimit.createRateLimitDirective({
  identifyContext: ctx => ctx.user.id,
  /**
   * Import the class from graphql-rate-limit and pass in an instance of redis client to the constructor
   */
  store: new graphqlRateLimit.RedisStore(redis.createClient())
});
```


