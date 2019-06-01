---
title: "Multiple Updates"
date: 2017-11-24T14:05:41-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 90      #rem
aliases: []
toc: false
draft: false
reviewed: true
---


Let's copy the queries to updates in order to work on updates.

```
cd ~/networknt/light-example-4j/rest/openapi/database
cp -r queries updates
```

Now let's update UpdatesGetHandler.java in the updates folder.

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

public class UpdatesGetHandler implements LightHttpHandler {

    private static final DataSource ds = SingletonServiceFactory.getBean(DataSource.class);
    private static final ObjectMapper mapper = Config.getInstance().getMapper();
    
    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        if (exchange.isInIoThread()) {
            exchange.dispatch(this);
            return;
        }
        int queries = Helper.getQueries(exchange);
        RandomNumber[] worlds = new RandomNumber[queries];
        try (final Connection connection = ds.getConnection()) {
            Map<Integer, Future<RandomNumber>> futureWorlds = new ConcurrentHashMap<>();
            for (int i = 0; i < queries; i++) {
                futureWorlds.put(i, Helper.EXECUTOR.submit(new Callable<RandomNumber>() {
                    @Override
                    public RandomNumber call() throws Exception {
                        RandomNumber rn;
                        try (PreparedStatement update = connection.prepareStatement(
                                "UPDATE world SET randomNumber = ? WHERE id= ?")) {
                            try (PreparedStatement query = connection.prepareStatement(
                                    "SELECT * FROM world WHERE id = ?",
                                    ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)) {

                                query.setInt(1, Helper.randomWorld());
                                ResultSet resultSet = query.executeQuery();
                                resultSet.next();
                                rn = new RandomNumber(
                                        resultSet.getInt("id"),
                                        resultSet.getInt("randomNumber"));
                            }
                            rn.setRandomNumber(Helper.randomWorld());
                            update.setInt(1, rn.getRandomNumber());
                            update.setInt(2, rn.getId());
                            update.executeUpdate();
                            return rn;
                        }
                    }
                }));
            }
            for (int i = 0; i < queries; i++) {
                worlds[i] = futureWorlds.get(i).get();
            }
        }
        exchange.getResponseHeaders().put(
                Headers.CONTENT_TYPE, "application/json");
        exchange.getResponseSender().send(mapper.writeValueAsString(worlds));
    }
}

```


Let's build and start the server.

```
cd ~/networknt/light-example-4j/rest/openapi/database/updates
mvn clean install exec:exec

```

Let's test it with one update.

```
curl -k https://localhost:8443/v1/updates
```

Result: 

```
[{"randomNumber":10,"id":9}]
```

Let's test it with multiple updates.

```
curl -k https://localhost:8443/v1/updates?queries=5
```

Result:

```
[{"randomNumber":8,"id":5},{"randomNumber":3,"id":1},{"randomNumber":4,"id":6},{"randomNumber":1,"id":7},{"randomNumber":5,"id":7}]
```

Now we have created three endpoints for Mysql database to do a single query, multiple queries, and multiple updates. The [next step][] we will switch to the Oracle database without changing the source code. 

[next step]: /tutorial/rest/openapi/database/oracle/
