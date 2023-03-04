---
title: "Generic Transformer"
date: 2023-03-03T15:01:21-05:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

### Overview

The [generic proxy transformer][] paired with the Light4J [yaml-rule][] engine, the request/response interceptor handlers allow for request/response body transformation. The supported transformation operations right now are as follows: JSON to SOAP Transformation, SOAP to JSON Transformation, Encoding Transformation and Field Text Formatting.


### How it Works

For all types of transformations that the generic transformer supports, we specify what we need in the rules.yml file. The transformation manager parses the configuration and creates a double-linked list of transformers. When the transformation process happens, the next transformer in line grabs the result of the previous and uses that result object as a base. This way, we can apply any transformations to a body without interference (with some exceptions).

The transformer base class that each transformation class extends has four important methods. Transformers follow the same pattern for the order in which each method is executed; the main difference between them is the transformation logic itself. The base and result objects are always in the LinkedHashMap structure (to avoid any unintended change in the order of keys). When we finalize the object in the last transformer, we serialize the object using Jackson. 


The following is a list of methods and descriptions


* init()

Initializes anything that the current transformer requires.

* doTransform()

Execute the transformation logic.

* getAsString()

Finalize the object, and get the final value in string format. This is only executed in the last transformer in the chain.

* passToNextTransformer()

Takes the end result of the current transformer and assigns the value as the base of the next transformer.


### How to Configure

Due to limitations of the rule engine, we currently have to create a stringified value as our config in rules.yml. For all transformers, the pattern for defining what transformers to be used are as follows:

* Declare the field you want to transform first.

* Separate each operation applied to the field with an '@@@'.

  - Operations are always in key-value pairs; they, too, must be separated by an '@@@'.

* Separate different fields with a ',' (comma)

* All together the pattern looks like this: <Field1>@@@<Operation/Transformation Type>@@@<Operation/Transformation Value>

Example: MyFirstField@@@$prefix@@@test

In Object: 

```
{
    MyFirstString: "someString"
    MyFirstField: {
        testString: "string"
        someBoolean: false
    }
}
```

Result: 

```
{
    MyFirstString: "someString"
    test:MyFirstField: {
        testString: "string"
        someBoolean: false
    }
}
```

### JSON to SOAP Transformer

This transformer allows the user to translate their requests or responses from JSON format to SOAP. Besides changing the data type of the body, the transformer also allows the user to add XML style attributes to specific fields. Keep in mind that this transformer will also place your body into a SOAP Envelope structure.


For example:
Example: MyFirstField@@@attributeName@@@attributeValue@@@$prefix@@@ea@@@xmlns:ea@@@myNameSpaceValue

In Object: 

```
{
    MyFirstString: "someString"
    MyFirstField: {
        testString: "string"
        someBoolean: false
    }
}
```

Result: 

```
<Envelope>
  </Header>
  <Body>
    <MyFirstString>someString</MyFirstString>
    <ea:MyFirstField attributeName=attributeValue xmlns:ea=myNameSpaceValue>
      <testString>string</testString>
      <someBoolean>false</someBoolean>
    </ea:MyFirstField>
  </Body>
</Envelope>
```

### SOAP to JSON Transformer

This operation is fairly straightforward; it allows the request or response to be translated from SOAP to JSON.

For example:

In Object: 

```
<Envelope>
  </Header>
  <Body>
    <MyFirstString>someString</MyFirstString>
    <ea:MyFirstField attributeName=attributeValue xmlns:ea=myNameSpaceValue>
      <testString>string</testString>
      <someBoolean>false</someBoolean>
    </ea:MyFirstField>
  </Body>
</Envelope>
```

Result: 

```
{
    MyFirstString: "someString"
    ea:MyFirstField: {
        testString: "string"
        someBoolean: false
    }
}
```

### Encoding Transformer

This operation allows the user to take their request or response body and encode the data (or encode a subsection of data).

For Example:

Example: MyFirstField@@@$encode@@@base64

In Object: 

```
{
    MyFirstString: "someString"
    MyFirstField: {
        testString: "string"
        someBoolean: false
    }
}
```

Result: 

```
{
    MyFirstString: "someString"
    MyFirstField: ewogICAgICAgIHRlc3RTdHJpbmc6ICJzdHJpbmciCiAgICAgICAgc29tZUJvb2xlYW46IGZhbHNlCiAgICB9
}
```

### Future Expansion/Development

##### Config String Changes

In the future, we should be able to accept more data types other than a string for the rules.yml config. This would make configuring the generic transformer an easier task and improve the general readability of these types of configurations. As an example, here is how we could represent key-value pair modifiers for a field:

```
MyFirstField:
    - xmlAttributes:
        - [fieldName, fieldValue]
        - [anotherFieldName, anotherFieldValue]
    - encode:
        - [fieldName, base64]
```

##### Create More/Expand Transformers

There are plenty of operations that we do not cover in the generic transformer. We can easily expand on any of them or create new ones altogether. For example, we could have a transformer to change the formatting of strings (all caps, all lowercase, etc.). Additionally, we can split off some of the features in the JSON to SOAP transformer to their own class. 


[yaml-rule]: https://github.com/networknt/yaml-rule
[generic proxy transformer]: https://github.com/networknt/yaml-rule-plugin/tree/master/generic-rest2soap
