---
title: "Multiple Queries"
date: 2017-11-24T13:20:57-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 80      #rem
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have updated the QueryGetHandler to retrieve data from the database for a single record. In this step, we are going to retrieve multiple records from the database. 

Let's build multiple queries based on the codebase of the single query.

```
cd ~/networknt/light-example-4j/rest/openapi/database
cp -r query queries
```

Now let's update queries project for QueriesGetHandler.java

```
package com.networknt.database.handler;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.config.Config;
import com.networknt.database.model.RandomNumber;
import com.networknt.handler.LightHttpHandler;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.Headers;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.Map;
import java.util.concurrent.Callable;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Future;

public class QueriesGetHandler implements LightHttpHandler {

    private static final DataSource ds = SingletonServiceFactory.getBean(DataSource.class);
    private static final ObjectMapper mapper = Config.getInstance().getMapper();

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        if (exchange.isInIoThread()) {
            exchange.dispatch(this);
            return;
        }
        int queries = Helper.getQueries(exchange);

        RandomNumber[] randomNumbers = new RandomNumber[queries];
        try (final Connection connection = ds.getConnection()) {
            Map<Integer, Future<RandomNumber>> futureWorlds = new ConcurrentHashMap<>();
            for (int i = 0; i < queries; i++) {
                futureWorlds.put(i, Helper.EXECUTOR.submit(new Callable<RandomNumber>(){
                    @Override
                    public RandomNumber call() throws Exception {
                        try (PreparedStatement statement = connection.prepareStatement(
                                "SELECT * FROM world WHERE id = ?",
                                ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)) {

                            statement.setInt(1, Helper.randomWorld());
                            ResultSet resultSet = statement.executeQuery();
                            resultSet.next();
                            return new RandomNumber(
                                    resultSet.getInt("id"),
                                    resultSet.getInt("randomNumber"));
                        }
                    }
                }));
            }

            for (int i = 0; i < queries; i++) {
                randomNumbers[i] = futureWorlds.get(i).get();
            }
        }
        exchange.getResponseHeaders().put(
                Headers.CONTENT_TYPE, "application/json");
        exchange.getResponseSender().send(mapper.writeValueAsString(randomNumbers));
     }
}

```

Now let's build and test the server

```
cd ~/networknt/light-example-4j/rest/openapi/database/queries
mvn clean install exec:exec
```

Let's test it.

```
curl -k https://localhost:8443/v1/queries
```

Result:

```
[{"randomNumber":3,"id":4}]
```

Again with 5 random numbers

```
curl -k https://localhost:8443/v1/queries?queries=5
```
Result: 

```
[{"randomNumber":2,"id":5},{"randomNumber":2,"id":1},{"randomNumber":8,"id":10},{"randomNumber":3,"id":6},{"randomNumber":2,"id":5}]
```

In the next step we are going to [update][] some records in database. 

[update]: /tutorial/rest/openapi/database/multiple-updates/