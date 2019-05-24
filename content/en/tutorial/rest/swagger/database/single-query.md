---
title: "Query DB"
date: 2017-11-24T13:18:03-05:00
description: ""
categories: []
keywords: []
slug: ""
weight: 70      #rem
aliases: []
toc: false
draft: false
reviewed: true
---

In the previous step, we have created a private static data source in each handler. In this step, we are going to use that connection pool to access the MySQL database. 

Before we add logic to query the database, let's copy connection to query folder.

```
cd ~/networknt/light-example-4j/rest/swagger/database
cp -r connection query

```

And add a constructor that accepts two integers as parameters for RandomNumber in model folder. This will ensure that it is easy to construct an object from QueryGetHandler. 

```
  public RandomNumber(int id, int randomNumber) {
    this.id = id;
    this.randomNumber = randomNumber;
  }

```

And add a helper class Helper.java to provide some utilities to be called from our handlers. It will make the handler code much simpler and clearer. 

```
package com.networknt.database.handler;

import io.undertow.server.HttpServerExchange;

import java.util.Deque;
import java.util.concurrent.*;

/**
 * Created by stevehu on 2017-01-23.
 */
public class Helper {
    private Helper() {
        throw new AssertionError();
    }

    /**
     * Returns the value of the "queries" request parameter, which is an integer
     * bound between 1 and 5 with a default value of 1.
     *
     * @param exchange the current HTTP exchange
     * @return the value of the "queries" request parameter
     */
    static int getQueries(HttpServerExchange exchange) {
        Deque<String> values = exchange.getQueryParameters().get("queries");
        if (values == null) {
            return 1;
        }
        String textValue = values.peekFirst();
        if (textValue == null) {
            return 1;
        }
        try {
            int parsedValue = Integer.parseInt(textValue);
            return Math.min(5, Math.max(1, parsedValue));
        } catch (NumberFormatException e) {
            return 1;
        }
    }

    /**
     * Returns a random integer that is a suitable value for both the {@code id}
     * and {@code randomNumber} properties of a world object.
     *
     * @return a random world number
     */
    static int randomWorld() {
        return 1 + ThreadLocalRandom.current().nextInt(10);
    }

    private static final int cpuCount = Runtime.getRuntime().availableProcessors();

    // todo: parameterize multipliers
    public static ExecutorService EXECUTOR =
            new ThreadPoolExecutor(
                    cpuCount * 2, cpuCount * 25, 200, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>(cpuCount * 100),
                    new ThreadPoolExecutor.CallerRunsPolicy());

}

```

Let's update QueryGetHandler.java

```
package com.networknt.database.handler;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.config.Config;
import com.networknt.database.model.RandomNumber;
import com.networknt.service.SingletonServiceFactory;
import io.undertow.server.HttpHandler;
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

public class QueryGetHandler implements HttpHandler {
    private static final DataSource ds = (DataSource) SingletonServiceFactory.getBean(DataSource.class);
    private static final ObjectMapper mapper = Config.getInstance().getMapper();

    @Override
    public void handleRequest(HttpServerExchange exchange) throws Exception {
        if (exchange.isInIoThread()) {
            exchange.dispatch(this);
            return;
        }
        int queries = 1;

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

        exchange.getResponseSender().send(mapper.writeValueAsString(randomNumbers[0]));
    }
}

```


We are good to go.

```
cd ~/networknt/light-example-4j/rest/swagger/database/query
mvn clean install exec:exec
```

Access the query endpoint, and you will get the random number as a result.

Note that we need to make sure MySQL database docker container is up and running. If you got an error in the console, chances are your MySQL database is not running. Please refer to
[start databases][] for instructions to start MySQL database in Docker. 


```
curl -k https://localhost:8443/v1/query
```

Result:

```
{"randomNumber":0,"id":2}
```

In the next step, we are going to update [multiple queries][] handler.

[start databases]: /tutorial/rest/swagger/database/startdb/
[multiple queries]: /tutorial/rest/swagger/database/multiple-queries/
