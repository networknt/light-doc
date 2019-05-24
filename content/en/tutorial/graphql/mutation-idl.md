---
title: "Mutation Tutorial with IDL"
date: 2018-01-03T10:31:30-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

In the [mutation tutorial][], we have create a new project by copying and modifying [hello world tutorial][].
In this tutorial, we are going to generate the project from IDL and modify it to build the same project. Users
can compare between these two implementation and see the difference. Also, this might help users to realize that
in which situation, you should just copy existing project and create the schema on your own or create the schema
first and generate the project from it. 

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

### Config and Schema

In order to make the two mutation project comparable, we are going to use the same config.json file.

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

The schema file mutation-schema.graphqls can be found at model-config/graphql/mutation-idl folder.

```
schema {
  query: Query
  mutation: Mutation
}

type Query {
  numberHolder: NumberHolder
}

type NumberHolder {
  theNumber: Int
}

type Mutation {
  changeTheNumber(newNumber: Int!): NumberHolder
  failToChangeTheNumber(newNumber: Int): NumberHolder
}

```

### Generate

Given the light-codegen is built, let's generate the project from the config file and IDL. For
more detail about the light-codegen please refer to [light-codegen graphql][].

We are going to generate the code to the same folder on light-example-4j/graphql/mutation-idl, let's rename 
the existing folder so that you can compare after you follow the tutorial. 

```
cd ~/networknt/light-example-4j/graphql
mv mutation-idl mutation-idl.bak
```

Now let's run the command line in ~/networknt folder.

```
cd ~/networknt
java -jar light-codegen/codegen-cli/target/codegen-cli.jar -f light-graphql-4j -o light-example-4j/graphql/mutation-idl -m model-config/graphql/mutation-idl/mutation-schema.graphqls -c model-config/graphql/mutation-idl/config.json

```


### Modify

After the light-codegen, we have the following StarWarsSchema generated.  

```java
package com.networknt.starwars.schema;

import com.networknt.graphql.router.SchemaProvider;
import graphql.schema.GraphQLSchema;
import graphql.schema.idl.RuntimeWiring;
import graphql.schema.idl.SchemaGenerator;
import graphql.schema.idl.SchemaParser;
import graphql.schema.idl.TypeDefinitionRegistry;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

/**
 * Created by steve on 25/03/17.
 */
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

Let's create a model and wire in the logic to make the generated code works. 

First let's create a model class for NumberHolder

```java
package com.networknt.starwars.schema;

/**
 * Created by Nicholas Azar on October 16, 2017.
 */
public class NumberHolder {
    private int theNumber;

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

```

Next let's create a class that wire the business logic.

```java
package com.networknt.starwars.schema;

import graphql.schema.DataFetcher;

/**
 * Created by Nicholas Azar on October 16, 2017.
 */
public class MutationWiring {

    public static class Context {
        final NumberHolder numberHolder;

        public Context(int theNumber) {
            this.numberHolder = new NumberHolder(theNumber);
        }

        public NumberHolder getNumberHolder() {
            return this.numberHolder;
        }
    }

    private static Context context = new Context(5);

    static DataFetcher numberHolderFetcher = dataFetchingEnvironment -> context.getNumberHolder();

    static DataFetcher theNumberFetcher = dataFetchingEnvironment -> context.getNumberHolder().getTheNumber();

    static DataFetcher changeTheNumberFetcher = dataFetchingEnvironment -> {
        context.getNumberHolder().setTheNumber(dataFetchingEnvironment.getArgument("newNumber"));
        return context.getNumberHolder();
    };

    static DataFetcher failToChangeTheNumberFetcher = dataFetchingEnvironment -> {
        throw new RuntimeException("Simulate failing to change the number.");
    };

}

```

With above two classes, let's modify the generated code to wire the logic in.

```java
package com.networknt.starwars.schema;

import com.networknt.graphql.router.SchemaProvider;
import graphql.schema.GraphQLSchema;
import graphql.schema.idl.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

/**
 * Created by Nicholas Azar on October 16, 2017.
 */
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

        RuntimeWiring wiring = buildRuntimeWiring();
        return new SchemaGenerator().makeExecutableSchema(typeRegistry, wiring);
    }

    private RuntimeWiring buildRuntimeWiring() {
        return RuntimeWiring.newRuntimeWiring()
                .type(TypeRuntimeWiring.newTypeWiring("Query")
                        .dataFetcher("numberHolder", MutationWiring.numberHolderFetcher))
                .type(TypeRuntimeWiring.newTypeWiring("theNumber")
                        .dataFetcher("theNumber", MutationWiring.theNumberFetcher))
                .type(TypeRuntimeWiring.newTypeWiring("Mutation")
                        .dataFetcher("changeTheNumber", MutationWiring.changeTheNumberFetcher)
                        .dataFetcher("failToChangeTheNumber", MutationWiring.failToChangeTheNumberFetcher)
                )
                .build();
    }

}

```

### Build

With all the changes done, we can build the server and start it.

```
cd ~/networknt/light-example-4j/graphql/mutation-idl
mvn clean install exec:exec
```


### Test

Now we can test it with GraphiQL web interface.

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



[mutation tutorial]: /tutorial/graphql/mutation/
[hello world tutorial]: /tutorial/graphql/helloworld/
[light-codegen graphql]: /tutorial/generator/graphql/
