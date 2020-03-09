# ksqldb-graphql
Graphql integration for ksqldb

# Background
Because of ksqldb [klip-15](https://github.com/confluentinc/ksql/pull/4069), there will have a new API with which to interact. This library contains graphql wrappers around that API in order to facilitate easier app creation.

# Installation
`yarn install @ksqldb/graphql`
 
# Integration
For statements to be executed against ksqdb, a `requester` must be provided, both at startup and in the context of the graphql resolvers. It is assumed that it has a signature the same to that of [axios](https://github.com/axios/axios).

The prebuild step, `buildKsqlDBGraphQL`, returns a promise and should be called prior to starting the graphql server.

# Usage

```js
import { connect } from 'http2';
import { ApolloServer } from 'apollo-server';
import { buildKsqlDBGraphQL } from '@ksqldb/graphql';
import { addResolveFunctionsToSchema } from 'graphql-tools';
import axios from 'axios';

const session = connect(`http://localhost:8089`);
const instance = axios.create({
  baseURL: ksqlServer,
  timeout: 1000,
});

buildKsqlDBGraphQL({ requester: instance }).then(
  ({ schemas, queryResolvers, subscriptionResolvers, mutationResolvers }) => {
    const apolloResolvers = {
      Subscription: subscriptionResolvers,
      Query: queryResolvers,
      Mutation: mutationResolvers,
    };
    const schema = addResolveFunctionsToSchema({ schema: schemas, resolvers: apolloResolvers });
    const server = new ApolloServer({
      context: async () => ({
        ksqlDB: {
          requester: instance,
          session,
        },
      }),
      schema,
      subscriptions: {
        keepAlive: 1000,
      },
    });
    server.listen();
  }
);
```


# Packages
### @ksqldb/graphql
The main graphql package. This generates the schema and resolvers based off of an existing ksqldb cluster.

### [Rideshare](https://github.com/confluentinc/ksqldb-graphql/tree/master/packages/rideshare)

