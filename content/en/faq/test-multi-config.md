---
title: "Test with different config files"
date: 2017-11-08T07:54:32-05:00
description: ""
categories: []
keywords: [test,unit]
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

As most modules in light-4j have a config file, there are different options available
to change the behaviour of the module through config. When writing unit tests, we want
to test the module behaviours in different configurations. 

The config module supports loading config file from module local, application, classpath
and system property specified directory. The following example will construct and load
different config files in each test class. 

The entire code can be found [status test][] folder

## Manually construct the config

The status module supports a customized serializer to format the status in a different format
suitable for each organization through service.yml dependency injection. If there is no
implementation specified or there is no service.yml available, then the default serializer
will be used to output the status object into a flattened map. 

Here is the test class with manually constructed service.json to specify StatusSerializer
implementation. 

```java
package com.networknt.status;

import com.networknt.config.Config;
import org.junit.AfterClass;
import org.junit.Assert;
import org.junit.BeforeClass;
import org.junit.Test;

import java.io.File;
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class StatusSerializerTest {

    static Config config = null;
    static final String homeDir = System.getProperty("user.home");

    @BeforeClass
    public static void setUp() throws Exception {
        config = Config.getInstance();

        // write a config file into the user home directory.
        List<String> implementationList = new ArrayList<>();
        implementationList.add("com.networknt.status.ErrorRootStatusSerializer");
        Map<String, List<String>> implementationMap = new HashMap<>();
        implementationMap.put("com.networknt.status.StatusSerializer", implementationList);
        List<Map<String, List<String>>> interfaceList = new ArrayList<>();
        interfaceList.add(implementationMap);
        Map<String, Object> singletons = new HashMap<>();
        singletons.put("singletons", interfaceList);
        config.getMapper().writeValue(new File(homeDir + "/service.json"), singletons);

        // Add home directory to the classpath of the system class loader.
        addURL(new File(homeDir).toURI().toURL());
    }

    @AfterClass
    public static void tearDown() throws Exception {
        // Remove the test.json from home directory
        File test = new File(homeDir + "/service.json");
        test.delete();
    }

    public void testGetConfigFromClassPath() throws Exception {
        Map<String, Object> configMap = config.getJsonMapConfig("service");
        Assert.assertNotNull(configMap.get("singletons"));
    }

    public static void addURL(URL url) throws Exception {
        URLClassLoader classLoader
                = (URLClassLoader) ClassLoader.getSystemClassLoader();
        Class clazz= URLClassLoader.class;

        // Use reflection
        Method method= clazz.getDeclaredMethod("addURL", URL.class);
        method.setAccessible(true);
        method.invoke(classLoader, url);
    }

    @Test
    public void testToString() {
        Status status = new Status("ERR10001");
        System.out.println(status);
        // test with ErrorStatusRootStatusSerializer
        Assert.assertEquals("{ \"error\" : {\"statusCode\":401,\"code\":\"ERR10001\",\"message\":\"AUTH_TOKEN_EXPIRED\",\"description\":\"Jwt token in authorization header expired\"} }", status.toString());
    }

    @Test
    public void testToStringWithArgs() {
        Status status = new Status("ERR11000", "parameter name", "original url");
        System.out.println(status);
        // test with ErrorStatusRootStatusSerializer
        Assert.assertEquals("{ \"error\" : {\"statusCode\":400,\"code\":\"ERR11000\",\"message\":\"VALIDATOR_REQUEST_PARAMETER_QUERY_MISSING\",\"description\":\"Query parameter parameter name is required on path original url but not found in request.\"} }", status.toString());
    }

}

```

As you can see, in the junit setUp static method, a json object is constructed to write to
service.json in the home directory of the current user. The last line adds this directory to
the classpath. 

During tearDown, the file written in the setUp is removed for clean up.

In this case, the two test cases will use the service.json config file which will load ErrorRootStatusSerializer
to serialize the status object with "error" as root object. 

## Run junit tests in separate classloaders

Now let's take a look at the StatusDefaultTest class. JUnit will use the same JVM to run all
the test classes in the same module. The previous test class has created a singleton instance
of config already. If we don't do anything, the same instance will be used. In order to switch to
the state that there is no service.yml config available. We need to have another classloader to
run this test case.

```java
package com.networknt.status;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.config.Config;
import org.junit.Assert;
import org.junit.BeforeClass;
import org.junit.Test;
import org.junit.runner.RunWith;

/**
 * Created by steve on 23/09/16.
 */
@RunWith(SeparateClassloaderTestRunner.class)
public class StatusDefaultTest {

    @Test
    public void testConstructor() {
        Status status = new Status("ERR10001");
        Assert.assertEquals(401, status.getStatusCode());
    }

    @Test
    public void testToString() {
        Status status = new Status("ERR10001");
        System.out.println(status);
        Assert.assertEquals("{\"statusCode\":401,\"code\":\"ERR10001\",\"message\":\"AUTH_TOKEN_EXPIRED\",\"description\":\"Jwt token in authorization header expired\"}", status.toString());
    }

    @Test
    public void testToStringWithArgs() {
        Status status = new Status("ERR11000", "parameter name", "original url");
        System.out.println(status);
        Assert.assertEquals("{\"statusCode\":400,\"code\":\"ERR11000\",\"message\":\"VALIDATOR_REQUEST_PARAMETER_QUERY_MISSING\",\"description\":\"Query parameter parameter name is required on path original url but not found in request.\"}", status.toString());
    }

    @Test
    public void testToStringPerf() {
        long start = System.currentTimeMillis();
        Status status = new Status("ERR10001");
        String s = null;
        for(int i = 0; i < 1000000; i++) {
            s = status.toString();
        }
        System.out.println("ToString Perf " + (System.currentTimeMillis() - start));
    }

    @Test
    public void testJacksonPerf() throws JsonProcessingException {
        ObjectMapper mapper = Config.getInstance().getMapper();
        long start = System.currentTimeMillis();
        Status status = new Status("ERR10001");
        String s = null;
        for(int i = 0; i < 1000000; i++) {
            s = mapper.writeValueAsString(status);
        }
        System.out.println("Jackson Perf " + (System.currentTimeMillis() - start));
    }
}

```

The RunWith annotation will use a customized ClassLoader to run this test. And the class loader
will ensure that all classes in package com.networknt. will be reloaded and hence the config file
will be reloaded. 

Here is the implementation of SeparateClassLoaderTestRunner

```java

package com.networknt.status;

import org.junit.runners.BlockJUnit4ClassRunner;
import org.junit.runners.model.InitializationError;

import java.net.URLClassLoader;

import static java.lang.ClassLoader.getSystemClassLoader;

public class SeparateClassloaderTestRunner  extends BlockJUnit4ClassRunner {

    public SeparateClassloaderTestRunner(Class<?> clazz) throws InitializationError {
        super(getFromTestClassloader(clazz));
    }

    private static Class<?> getFromTestClassloader(Class<?> clazz) throws InitializationError {
        try {
            ClassLoader testClassLoader = new TestClassLoader();
            return Class.forName(clazz.getName(), true, testClassLoader);
        } catch (ClassNotFoundException e) {
            throw new InitializationError(e);
        }
    }

    public static class TestClassLoader extends URLClassLoader {
        public TestClassLoader() {
            super(((URLClassLoader)getSystemClassLoader()).getURLs());
        }

        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            if (name.startsWith("com.networknt.")) {
                return super.findClass(name);
            }
            return super.loadClass(name);
        }
    }
}

```

[status test]: https://github.com/networknt/light-4j/tree/master/status/src/test/java/com/networknt/status
 