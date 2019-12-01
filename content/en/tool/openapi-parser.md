---
title: "OpenAPI Parser"
date: 2017-11-22T22:55:04-05:00
description: ""
categories: []
keywords: []
aliases: []
toc: false
draft: false
---

The OpenApi Parser is based on the generated code of KaiZen OpenApi Parser from RepreZen which is a Java-based validating parser for OpenAPI 3.0 offering full compliance with the [OpenAPI 3.0 Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md), and a highly uniform read/write programming API. [OpenAPI](http://openapis.org), formerly known as the Swagger specification is the industry-standard format for machine-readable REST API descriptions.

By removing the generator from KaiZen parser, this library doesn't depend on any external dependencies, which make it a green library suitable for microservices. 


Feature highlights of OpenAPI Parser include:

* **High Performance** - Informal testing shows a 3x-4x performance improvement over the current Java Swagger 2.0 parser. This is largely attributable to a design based on adapting Jackson
  `JsonNode` objects, rather than deserializing to internal POJOs.
  
* **Read/Write API** - All aspects of a model may be interrogated and modified. We also provide fluent builders for all model object types. We provide bidirectional navigation throughout the model, and every object that is a property value of its containing object (whether as a named field or a map entry) knows its own key.
  
* **Tolerant Reader** - The parser yields a fully accessible result from any valid JSON or YAML input - whether or not the input is a valid OpenAPI specification.
  
* **Separate, extensible validation** - All validation beyond base JSON/YAML parsing is performed after the initial parse, and it can be disabled for speed. Model-level validation (i.e. anything that can be checked with our provided Java API) is separated from injectable implementation-level validation. The latter covers anything that needs to look "under the covers", using methods of the implementation classes that lie outside the model API. For example, this is where unrecognized JSON properties would be noted and flagged, as they are not visible from the model API.

* **Serialization** - Serialization to JSON or YAML is supported, and by default, round-tripping will not cause any reordering of model content.
  
* **DI-Ready** - Our API is interface-based and allows substitution of custom implementations that either augment or completely replace our provided implementation, using [Singleton Service Factory][] dependency injection.
  
* **Green** - This library doesn't depend on any external dependencies which makes it light-weight and suitable for microservices. 
  
* **Flexible Reference Handling** - References are detected and resolved after JSON/YAML parsing but before model-level parsing. All JSON references are processed, resulting in effective inlining for non-conforming references (i.e. anything other than path or component references specifically allowed in the
specification). References are normally traversed automatically by the API, but full details of references and their resolution status are also available.


### Getting Started

You can try the parser with the following simple program, or of course, explore with your own models.

The program parses each of the examples OpenAPI 3.0 models currently available in the OAI/OpenAPI-Specification GitHub Repo. In each case, if validation succeeds, a summary of all the model's paths, operations, and operation parameters is printed. Otherwise, all validation messages are printed.

At the time of this writing, validation fails on the callback-example because that example does not include the required openapi and info properties.

```
package com.networknt.oas;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.networknt.oas.model.OpenApi3;
import com.networknt.oas.validator.ValidationResults.ValidationItem;
import com.networknt.utility.NioUtils;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameter;
import org.junit.runners.Parameterized.Parameters;

import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.util.*;

@RunWith(Parameterized.class)
public class ExamplesTest extends Assert {

	private static final String SPEC_REPO = "OAI/OpenAPI-specification";
	private static final String EXAMPLES_BRANCH = "master";
	private static final String EXAMPLES_ROOT = "examples/v3.0";

	private static ObjectMapper mapper = new ObjectMapper();

	@Parameters(name = "{index}: {1}")
	public static Collection<Object[]> findExamples() throws IOException {
		Collection<Object[]> examples = new ArrayList<>();
		Deque<URL> dirs = new ArrayDeque<>();
		String auth = System.getenv("GITHUB_AUTH") != null ? System.getenv("GITHUB_AUTH") + "@" : "";
		String request = String.format("https://%sapi.github.com/repos/%s/contents/%s?ref=%s", auth, SPEC_REPO,
				EXAMPLES_ROOT, EXAMPLES_BRANCH);
		dirs.add(new URL(request));
		while (!dirs.isEmpty()) {
			URL url = dirs.remove();
			try (InputStream is = url.openStream()) {
				String json = NioUtils.toString(is);
				JsonNode tree = mapper.readTree(json);
				for (JsonNode result : iterable(tree.elements())) {
					String type = result.get("type").asText();
					String path = result.get("path").asText();
					String resultUrl = result.get("url").asText();
					if (type.equals("dir")) {
						dirs.add(new URL(resultUrl));
					} else if (type.equals("file") && (path.endsWith(".yaml") || path.endsWith(".json"))) {
						String downloadUrl = result.get("download_url").asText();
						examples.add(new Object[] { new URL(downloadUrl), result.get("name").asText() });
					}
				}
			}
		}
		return examples;
	}

	@Parameter
	public URL exampleUrl;

	@Parameter(1)
	public String fileName;

	@Test
	public void exampleCanBeParsed() throws IOException {
		if (!exampleUrl.toString().contains("callback-example")) {
			OpenApi3 model = (OpenApi3) new OpenApiParser().parse(exampleUrl);
			for (ValidationItem item : model.getValidationItems()) {
				System.out.println(item);
			}
			assertTrue("Example was not valid: " + exampleUrl, model.isValid());
		}
	}

	private static <T> Iterable<T> iterable(final Iterator<T> iterator) {
		return new Iterable<T>() {
			@Override
			public Iterator<T> iterator() {
				return iterator;
			}
		};
	}
}

```


### API Overview

To be completed.

### Resources
* Blog Post: [Introducing KaiZen OpenAPI 3.0 Parser: fast, flexible Java parsing & validation](http://www.reprezen.com/blog/kaizen-openapi-3_0-parser-swagger-java-open-source)
* [KaiZen OpenAPI 3.0 Parser](https://github.com/RepreZen/KaiZen-OpenApi-Parser)


[Singleton Service Factory]: /concern/service/