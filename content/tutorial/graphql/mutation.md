---
title: "Mutation Tutorial"
date: 2018-01-03T10:31:21-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

Mutation is modified from [hello world][] example and some of the files are updated. The first step
is to make a copy of helloworld example in light-example-4j and then we can update several files to
have a new schema and wired in. 

### Prepare environment

First we need to clone light-codegen, model-config and light-example-4j into your workspace. 
I am using networknt as workspace in my home directory.

```
cd ~/networknt
git clone https://github.com/networknt/light-codegen.git
git clone https://github.com/networknt/light-example-4j.git
git clone https://github.com/networknt/model-config.git
``` 

Let's build light-codegen so that we can use the command line to generate the code.

```
cd ~/networknt/light-codegen
mvn clean install -DskipTests
```

### Copy from helloworld

First let's backup the existing mutation project in light-example-4j/graphql folder.  

```
cd ~/networknt/light-example-4j/graphql
mv mutation mutation.bak
``` 

Now let's copy the helloworld project to mutation. 

```
cd ~/networknt/light-example-4j/graphql
cp -r helloworld mutation
```

### Update

Now let's open the light-example-4j/graphql/mutation project and update some files. You can update 
pom.xml to change the project group and artifact but here we will just leave it as it is. 


##### MutationSchema.java

Now let's create a new MutationSchema class as following in com.networknt.starwars.schema folder and
remove the existing StarWarsSchema.java 


```
package com.networknt.starwars.schema;

import com.networknt.graphql.router.SchemaProvider;
import graphql.schema.DataFetcher;
import graphql.schema.DataFetchingEnvironment;
import graphql.schema.GraphQLObjectType;
import graphql.schema.GraphQLSchema;

import static graphql.Scalars.GraphQLInt;
import static graphql.schema.GraphQLArgument.newArgument;
import static graphql.schema.GraphQLFieldDefinition.newFieldDefinition;

public class MutationSchema  implements SchemaProvider {
    public static class NumberHolder {
        int theNumber;

        public NumberHolder(int theNumber) {
            this.theNumber = theNumber;
        }

        public int getTheNumber() {
            return theNumber;
        }

        public void setTheNumber(int theNumber) {
            this.theNumber = theNumber;
        }


    }

    public static class Root {
        NumberHolder numberHolder;

        public Root(int number) {
            this.numberHolder = new NumberHolder(number);
        }

        public NumberHolder changeNumber(int newNumber) {
            this.numberHolder.theNumber = newNumber;
            return this.numberHolder;
        }


        public NumberHolder failToChangeTheNumber(int newNumber) {
            throw new RuntimeException("Cannot change the number");
        }
    }

    public static Root root = new Root(6);

    public static GraphQLObjectType numberHolderType = GraphQLObjectType.newObject()
            .name("NumberHolder")
            .field(newFieldDefinition()
                    .name("theNumber")
                    .type(GraphQLInt))
            .build();

    public static GraphQLObjectType queryType = GraphQLObjectType.newObject()
            .name("queryType")
            .field(newFieldDefinition()
                    .name("numberHolder")
                    .dataFetcher(new DataFetcher() {
                        @Override
                        public Object get(DataFetchingEnvironment environment) {
                            return root.numberHolder;
                        }
                    })
                    .type(numberHolderType))
            .build();

    public static GraphQLObjectType mutationType = GraphQLObjectType.newObject()
            .name("mutationType")
            .field(newFieldDefinition()
                    .name("changeTheNumber")
                    .type(numberHolderType)
                    .argument(newArgument()
                            .name("newNumber")
                            .type(GraphQLInt))
                    .dataFetcher(new DataFetcher() {
                        @Override
                        public Object get(DataFetchingEnvironment environment) {
                            Integer newNumber = environment.getArgument("newNumber");
                            return root.changeNumber(newNumber);
                        }
                    }))
            .field(newFieldDefinition()
                    .name("failToChangeTheNumber")
                    .type(numberHolderType)
                    .argument(newArgument()
                            .name("newNumber")
                            .type(GraphQLInt))
                    .dataFetcher(new DataFetcher() {
                        @Override
                        public Object get(DataFetchingEnvironment environment) {
                            Integer newNumber = environment.getArgument("newNumber");
                            return root.failToChangeTheNumber(newNumber);
                        }
                    }))
            .build();

    @Override
    public GraphQLSchema getSchema() {
        return GraphQLSchema.newSchema()
                .query(queryType)
                .mutation(mutationType)
                .build();
    }

}

```

##### service.yml

Now let's update service.yml to replace StarWarsSchema with MutationSchema for the SchemaProvider
interface. 

```yaml

# Singleton service factory configuration/IoC injection
singletons:
# HandlerProvider implementation
- com.networknt.server.HandlerProvider:
  - com.networknt.graphql.router.GraphqlRouter
# StartupHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.StartupHookProvider:
  # - com.networknt.server.Test1StartupHook
  # - com.networknt.server.Test1StartupHook
# ShutdownHookProvider implementations, there are one to many and they are called in the same sequence defined.
# - com.networknt.server.ShutdownHookProvider:
  # - com.networknt.server.Test1ShutdownHook
# MiddlewareHandler implementations, the calling sequence is as defined in the request/response chain.
- com.networknt.handler.MiddlewareHandler:
  # Exception Global exception handler that needs to be called first to wrap all middleware handlers and business handlers
  - com.networknt.exception.ExceptionHandler
  # Metrics handler to calculate response time accurately, this needs to be the second handler in the chain.
  - com.networknt.metrics.MetricsHandler
  # Traceability Put traceabilityId into response header from request header if it exists
  - com.networknt.traceability.TraceabilityHandler
  # Correlation Create correlationId if it doesn't exist in the request header and put it into the request header
  - com.networknt.correlation.CorrelationHandler
  # Security JWT token verification and scope verification for GraphQL
  - com.networknt.graphql.security.JwtVerifyHandler
  # SimpleAudit Log important info about the request into audit log
  - com.networknt.audit.AuditHandler
  # Validator Validate request based on graphql schema
  - com.networknt.graphql.validator.ValidatorHandler
# GraphQL schema provider interface implementation
- com.networknt.graphql.router.SchemaProvider:
  - com.networknt.starwars.schema.MutationSchema


- javax.sql.DataSource:
  - com.zaxxer.hikari.HikariDataSource:
      DriverClassName: com.mysql.jdbc.Driver
      jdbcUrl: jdbc:mysql://mysqldb:3306/oauth2?useSSL=false
      username: root
      password: my-secret-pw
      maximumPoolSize: 10
      useServerPrepStmts: true,
      cachePrepStmts: true,
      cacheCallableStmts: true,
      prepStmtCacheSize: 10,
      prepStmtCacheSqlLimit: 2048,
      connectionTimeout: 2000


```


### Build and Start server 

```
cd ~/networknt/light-example-4j/graphql/mutation
mvn clean install exec:exec
```

### Test

Open your browser and point to 

```
http://localhost:8080/graphql
```

Now you can explore the schema on Documentation Explorer. There should be a query and a mutation.

* Test query with GraphiQL

```
query {
  numberHolder {
    theNumber
  }
}
```

The result should be something like:

```json
{
  "data": {
    "numberHolder": {
      "theNumber": 6
    }
  }
}
```

* Test mutation with GraphiQL

```
mutation {
  changeTheNumber(newNumber:4) {
    theNumber
  }
}
```

The result should be: 

```json
{
  "data": {
    "changeTheNumber": {
      "theNumber": 4
    }
  }
}
```

* Test the query again

With the same query in the beginning, we should have the updated value 4 returned. 

```
query {
  numberHolder {
    theNumber
  }
}

```

The result should be: 

```json
{
  "data": {
    "numberHolder": {
      "theNumber": 4
    }
  }
}
```

* Test mutation with variables

Query: 

```
mutation ($n: Int!) {
  changeTheNumber(newNumber: $n) {
    theNumber
  }
}

```
Variables
```
{"n": 5}
```

And the result should be: 

```json
{
  "data": {
    "changeTheNumber": {
      "theNumber": 5
    }
  }
}
```

You can also use the following curl command to test.

```
curl -H 'Content-Type:application/json' -XPOST http://localhost:8080/graphql -d '{"query":"{ numberHolder { theNumber }}"}'

```

[hello world]: /tutorial/graphql/helloworld/
