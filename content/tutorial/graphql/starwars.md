---
title: "Starwars Tutorial"
date: 2018-01-03T12:10:51-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

This tutorial is very similar with [Hello World][]. The only difference is that this one is
generated from the star wars schema. 

Schema is defined in a file with extension .graphqls and the schema used in this project can
be found at https://github.com/networknt/model-config/blob/master/graphql/starwars/schema.graphqls

Here is the content of the schema.

```
schema {
    query: QueryType
}

type QueryType {
    hero(episode: Episode): Character
    human(id : String) : Human
    droid(id: ID!): Droid
}


enum Episode {
    NEWHOPE
    EMPIRE
    JEDI
}

interface Character {
    id: ID!
    name: String!
    friends: [Character]
    appearsIn: [Episode]!
}

type Human implements Character {
    id: ID!
    name: String!
    friends: [Character]
    appearsIn: [Episode]!
    homePlanet: String
}

type Droid implements Character {
    id: ID!
    name: String!
    friends: [Character]
    appearsIn: [Episode]!
    primaryFunction: String
}
```

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

### Config

Let's take a look at the config.json that control how the project is going to be generated.
It can be found at https://github.com/networknt/model-config/blob/master/graphql/starwars/config.json

Here is the content.


```json
{
  "name": "starwars",
  "version": "1.0.1",
  "groupId": "com.networknt",
  "artifactId": "starwars",
  "schemaPackage": "com.networknt.starwars.schema",
  "schemaClass": "StarWarsSchema",
  "overwriteSchemaClass": true,
  "httpPort": 8080,
  "enableHttp": true,
  "httpsPort": 8443,
  "enableHttps": false,
  "enableRegistry": false,
  "supportDb": true,
  "dbInfo": {
    "name": "mysql",
    "driverClassName": "com.mysql.jdbc.Driver",
    "jdbcUrl": "jdbc:mysql://mysqldb:3306/oauth2?useSSL=false",
    "username": "root",
    "password": "my-secret-pw"
  },
  "supportH2ForTest": false,
  "supportClient": false
}
```

### Generate

Given the light-codegen is built, let's generate the project from the config file only without IDL. For
more detail about the light-codegen please refer to [light-codegen graphql][].

We are going to generate the code to the same folder on light-example-4j/graphql/starwars, let's rename the
existing folder so that you can compare after you follow the tutorial. 

```
cd ~/networknt/light-example-4j/graphql
mv starwars starwars.bak
```

Now let's run the command line in ~/networknt folder.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-graphql-4j -o light-example-4j/graphql/starwars -m model-config/graphql/starwars/schema.graphqls -c model-config/graphql/starwars/config.json

```

### Build

The server can be built but cannot be started as there is no business logic wired in in the generated
code.  

```
cd ~/networknt/light-example-4j/graphql/stawars
mvn clean install

```


### Code

As we are using schema to generate the code and there is no business logic, the server cannot be started
although it can be built.

Here is the StarWarsSchema that implements SchemaProvider interface. 

```java
public class StarWarsSchema implements SchemaProvider {
    private static Logger logger = LoggerFactory.getLogger(SchemaProvider.class);
    private static String schemaName = "schema.graphqls";
    @Override
    public GraphQLSchema getSchema() {
        SchemaParser schemaParser = new SchemaParser();
        TypeDefinitionRegistry typeRegistry = null;

        try(InputStream is = getClass().getClassLoader().getResourceAsStream(schemaName)) {
            typeRegistry = schemaParser.parse(new InputStreamReader(is));
        } catch (IOException e) {
            logger.error("IOException:", e);
        }

        RuntimeWiring wiring = RuntimeWiring.newRuntimeWiring()
            // put other wiring logic here.
            .build();

        return new SchemaGenerator().makeExecutableSchema(typeRegistry, wiring);
    }
}

``` 

As you can see, I have put a comment "// put other wiring logic here." to indicate where you can put your
business logic in. 

The generator put the schema.graphqls file under src/main/resources and it will be used by the light-graphql-4j
framework during runtime as shown above. 

There is another config file service.yml that is very important to wire everything together. 

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
  - com.networknt.starwars.schema.StarWarsSchema


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

In the above generated file, the StarWarsSchema is injected to SchemaProvider interface. And there are several
light-graphql-4j and light-4j middleware handlers are wired into the request/response chain.


[Hello World]: /tutorial/graphql/helloworld/
[light-codegen graphql]: /tutorial/generator/graphql/
