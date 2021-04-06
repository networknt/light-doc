---
title: "Server Info"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---

### Introduction

Almost every module in light-4j has a configuration file externalized with default in the module itself or the API implementation config folder. A special handler injected in your hander.yml gives an overview of the server runtime, system properties, specification, and configurations for each enabled module. Once this handler endpoint is called, it will output all the server info in a JSON format. 


### Configuration

info.yml
```
# Server info endpoint that can output environment and component along with configuration

# Indicate if the server info is enable or not.
enableServerInfo: ${info.enableServerInfo:true}
```

Unlike other modules, the server info handler is plugged into the handler chain in the handler.yml file. It should not be part of the OpenAPI specification or other specifications for other frameworks if the API is not Restful.  If enableServerInfo is false, then an error message will be returned with ERR10013 - SERVER_INFO_DISABLED.

handler.yml

```
handlers:
  - com.networknt.info.ServerInfoGetHandler@info

.
.
.

paths:
  - path: '/server/info'
    method: 'get'
    exec:
      - security
      - info

```

The above path /server/info is the standard path that is used by the light-controller and light-portal for the runtime dashboard to access the server info after a service registers itself.  

Unlike the health check endpoint, the server info endpoint returns a lot of information for the server instance. It should be protected by the security handler all the time. When the service is registered to the light-controller, either standard edition running in a docker container or an enterprise edition deployed with light-portal, this endpoint should only be accessed with a special bootstrap token provided by light-controller.

### Extension

For other contributed modules or API application-specific modules, please follow the following guideline to register your module in /server/info endpoint.

For handlers, it is registered when injecting into the handler chain during server startup. For other utilities, it should have a static block to register itself during server startup.

Here is the example code to register a module and its config.

```
ModuleRegistry.registerModule(ValidatorHandler.class.getName(), Config.getInstance().getJsonMapConfigNoCache(ValidatorHandler.CONFIG_NAME), null);
```

Take a look at the handlers in light-4j for detailed implementation of middleware handlers. 

### Mask sensitive data in config

When a module registers itself, it provides the configuration in JSON format for the module. Some components have sensitive info in their configuration, for example, DB password, client secret etc. The third parameter in the registerModule method is a list of keys in the configuration file that need to be masked.

Here is an example.

```
List<String> masks = new ArrayList<String>();
masks.add("trustPass");
masks.add("keyPass");
ModuleRegistry.registerModule(Client.class.getName(), Config.getInstance().getJsonMapConfigNoCache(CONFIG_NAME), masks);

```

### Light Proxy

When the info handler is enabled on the light-proxy instance in a sidecar in a Kubernetes cluster, there are two scenarios for the response. 

If the backend API doesn't have the info implemented, then the proxy will return the response based on the proxy instance's collection. 

If the backend API has an info endpoint /server/info implemented, then the proxy info endpoint should combine both the proxy instance and backend API instance info. This is a separate implementation in the light-proxy project. 

### Output

Here is the output from the petstore API generated with the petstore OpenAPI specification.

```
{
  "environment": {
    "host": {
      "ip": "127.0.0.1",
      "hostname": "joy",
      "dns": "localhost"
    },
    "runtime": {
      "availableProcessors": 4,
      "freeMemory": 430837608,
      "totalMemory": 504889344,
      "maxMemory": 7486832640
    },
    "system": {
      "javaVendor": "Oracle Corporation",
      "javaVersion": "1.8.0_66",
      "osName": "Linux",
      "osVersion": "4.2.0-42-generic",
      "userTimezone": "America/Toronto"
    }
  },
  "specification": {
    "basePath": "/v2",
    "paths": {
      "/pet": {
        "post": {
          "tags": [
            "pet"
          ],
          "summary": "Add a new pet to the store",
          "description": "",
          "operationId": "addPet",
          "consumes": [
            "application/json",
            "application/xml"
          ],
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "in": "body",
              "name": "body",
              "description": "Pet object that needs to be added to the store",
              "required": true,
              "schema": {
                "$ref": "#/definitions/Pet"
              }
            }
          ],
          "responses": {
            "405": {
              "description": "Invalid input"
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        },
        "put": {
          "tags": [
            "pet"
          ],
          "summary": "Update an existing pet",
          "description": "",
          "operationId": "updatePet",
          "consumes": [
            "application/json",
            "application/xml"
          ],
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "in": "body",
              "name": "body",
              "description": "Pet object that needs to be added to the store",
              "required": true,
              "schema": {
                "$ref": "#/definitions/Pet"
              }
            }
          ],
          "responses": {
            "400": {
              "description": "Invalid ID supplied"
            },
            "404": {
              "description": "Pet not found"
            },
            "405": {
              "description": "Validation exception"
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/pet/findByStatus": {
        "get": {
          "tags": [
            "pet"
          ],
          "summary": "Finds Pets by status",
          "description": "Multiple status values can be provided with comma separated strings",
          "operationId": "findPetsByStatus",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "status",
              "in": "query",
              "description": "Status values that need to be considered for filter",
              "required": true,
              "type": "array",
              "items": {
                "type": "string",
                "enum": [
                  "available",
                  "pending",
                  "sold"
                ],
                "default": "available"
              },
              "collectionFormat": "multi"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/definitions/Pet"
                }
              }
            },
            "400": {
              "description": "Invalid status value"
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/pet/findByTags": {
        "get": {
          "tags": [
            "pet"
          ],
          "summary": "Finds Pets by tags",
          "description": "Muliple tags can be provided with comma separated strings. Use tag1, tag2, tag3 for testing.",
          "operationId": "findPetsByTags",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "tags",
              "in": "query",
              "description": "Tags to filter by",
              "required": true,
              "type": "array",
              "items": {
                "type": "string"
              },
              "collectionFormat": "multi"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/definitions/Pet"
                }
              }
            },
            "400": {
              "description": "Invalid tag value"
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "deprecated": true,
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/pet/{petId}": {
        "get": {
          "tags": [
            "pet"
          ],
          "summary": "Find pet by ID",
          "description": "Returns a single pet",
          "operationId": "getPetById",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "petId",
              "in": "path",
              "description": "ID of pet to return",
              "required": true,
              "type": "integer",
              "format": "int64"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "$ref": "#/definitions/Pet"
              }
            },
            "400": {
              "description": "Invalid ID supplied"
            },
            "404": {
              "description": "Pet not found"
            }
          },
          "security": [
            {
              "api_key": []
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        },
        "post": {
          "tags": [
            "pet"
          ],
          "summary": "Updates a pet in the store with form data",
          "description": "",
          "operationId": "updatePetWithForm",
          "consumes": [
            "application/x-www-form-urlencoded"
          ],
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "petId",
              "in": "path",
              "description": "ID of pet that needs to be updated",
              "required": true,
              "type": "integer",
              "format": "int64"
            },
            {
              "name": "name",
              "in": "formData",
              "description": "Updated name of the pet",
              "required": false,
              "type": "string"
            },
            {
              "name": "status",
              "in": "formData",
              "description": "Updated status of the pet",
              "required": false,
              "type": "string"
            }
          ],
          "responses": {
            "405": {
              "description": "Invalid input"
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/x-www-form-urlencoded"
        },
        "delete": {
          "tags": [
            "pet"
          ],
          "summary": "Deletes a pet",
          "description": "",
          "operationId": "deletePet",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "api_key",
              "in": "header",
              "required": false,
              "type": "string"
            },
            {
              "name": "petId",
              "in": "path",
              "description": "Pet id to delete",
              "required": true,
              "type": "integer",
              "format": "int64"
            }
          ],
          "responses": {
            "400": {
              "description": "Invalid ID supplied"
            },
            "404": {
              "description": "Pet not found"
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/pet/{petId}/uploadImage": {
        "post": {
          "tags": [
            "pet"
          ],
          "summary": "uploads an image",
          "description": "",
          "operationId": "uploadFile",
          "consumes": [
            "multipart/form-data"
          ],
          "produces": [
            "application/json"
          ],
          "parameters": [
            {
              "name": "petId",
              "in": "path",
              "description": "ID of pet to update",
              "required": true,
              "type": "integer",
              "format": "int64"
            },
            {
              "name": "additionalMetadata",
              "in": "formData",
              "description": "Additional data to pass to server",
              "required": false,
              "type": "string"
            },
            {
              "name": "file",
              "in": "formData",
              "description": "file to upload",
              "required": false,
              "type": "file"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "$ref": "#/definitions/ApiResponse"
              }
            }
          },
          "security": [
            {
              "petstore_auth": [
                "write:pets",
                "read:pets"
              ]
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "multipart/form-data"
        }
      },
      "/store/inventory": {
        "get": {
          "tags": [
            "store"
          ],
          "summary": "Returns pet inventories by status",
          "description": "Returns a map of status codes to quantities",
          "operationId": "getInventory",
          "produces": [
            "application/json"
          ],
          "parameters": [],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "type": "object",
                "additionalProperties": {
                  "type": "integer",
                  "format": "int32"
                }
              }
            }
          },
          "security": [
            {
              "api_key": []
            }
          ],
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/store/order": {
        "post": {
          "tags": [
            "store"
          ],
          "summary": "Place an order for a pet",
          "description": "",
          "operationId": "placeOrder",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "in": "body",
              "name": "body",
              "description": "order placed for purchasing the pet",
              "required": true,
              "schema": {
                "$ref": "#/definitions/Order"
              }
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "$ref": "#/definitions/Order"
              }
            },
            "400": {
              "description": "Invalid Order"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/store/order/{orderId}": {
        "get": {
          "tags": [
            "store"
          ],
          "summary": "Find purchase order by ID",
          "description": "For valid response try integer IDs with value >= 1 and <= 10. Other values will generated exceptions",
          "operationId": "getOrderById",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "orderId",
              "in": "path",
              "description": "ID of pet that needs to be fetched",
              "required": true,
              "type": "integer",
              "maximum": 10,
              "minimum": 1,
              "format": "int64"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "$ref": "#/definitions/Order"
              }
            },
            "400": {
              "description": "Invalid ID supplied"
            },
            "404": {
              "description": "Order not found"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        },
        "delete": {
          "tags": [
            "store"
          ],
          "summary": "Delete purchase order by ID",
          "description": "For valid response try integer IDs with positive integer value. Negative or non-integer values will generate API errors",
          "operationId": "deleteOrder",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "orderId",
              "in": "path",
              "description": "ID of the order that needs to be deleted",
              "required": true,
              "type": "integer",
              "minimum": 1,
              "format": "int64"
            }
          ],
          "responses": {
            "400": {
              "description": "Invalid ID supplied"
            },
            "404": {
              "description": "Order not found"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/user": {
        "post": {
          "tags": [
            "user"
          ],
          "summary": "Create user",
          "description": "This can only be done by the logged in user.",
          "operationId": "createUser",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "in": "body",
              "name": "body",
              "description": "Created user object",
              "required": true,
              "schema": {
                "$ref": "#/definitions/User"
              }
            }
          ],
          "responses": {
            "default": {
              "description": "successful operation"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/user/createWithArray": {
        "post": {
          "tags": [
            "user"
          ],
          "summary": "Creates list of users with given input array",
          "description": "",
          "operationId": "createUsersWithArrayInput",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "in": "body",
              "name": "body",
              "description": "List of user object",
              "required": true,
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/definitions/User"
                }
              }
            }
          ],
          "responses": {
            "default": {
              "description": "successful operation"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/user/createWithList": {
        "post": {
          "tags": [
            "user"
          ],
          "summary": "Creates list of users with given input array",
          "description": "",
          "operationId": "createUsersWithListInput",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "in": "body",
              "name": "body",
              "description": "List of user object",
              "required": true,
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/definitions/User"
                }
              }
            }
          ],
          "responses": {
            "default": {
              "description": "successful operation"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/user/login": {
        "get": {
          "tags": [
            "user"
          ],
          "summary": "Logs user into the system",
          "description": "",
          "operationId": "loginUser",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "username",
              "in": "query",
              "description": "The user name for login",
              "required": true,
              "type": "string"
            },
            {
              "name": "password",
              "in": "query",
              "description": "The password for login in clear text",
              "required": true,
              "type": "string"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "type": "string"
              },
              "headers": {
                "X-Rate-Limit": {
                  "type": "integer",
                  "format": "int32",
                  "description": "calls per hour allowed by the user"
                },
                "X-Expires-After": {
                  "type": "string",
                  "format": "date-time",
                  "description": "date in UTC when token expires"
                }
              }
            },
            "400": {
              "description": "Invalid username/password supplied"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/user/logout": {
        "get": {
          "tags": [
            "user"
          ],
          "summary": "Logs out current logged in user session",
          "description": "",
          "operationId": "logoutUser",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [],
          "responses": {
            "default": {
              "description": "successful operation"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      },
      "/user/{username}": {
        "get": {
          "tags": [
            "user"
          ],
          "summary": "Get user by user name",
          "description": "",
          "operationId": "getUserByName",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "username",
              "in": "path",
              "description": "The name that needs to be fetched. Use user1 for testing. ",
              "required": true,
              "type": "string"
            }
          ],
          "responses": {
            "200": {
              "description": "successful operation",
              "schema": {
                "$ref": "#/definitions/User"
              }
            },
            "400": {
              "description": "Invalid username supplied"
            },
            "404": {
              "description": "User not found"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        },
        "put": {
          "tags": [
            "user"
          ],
          "summary": "Updated user",
          "description": "This can only be done by the logged in user.",
          "operationId": "updateUser",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "username",
              "in": "path",
              "description": "name that need to be updated",
              "required": true,
              "type": "string"
            },
            {
              "in": "body",
              "name": "body",
              "description": "Updated user object",
              "required": true,
              "schema": {
                "$ref": "#/definitions/User"
              }
            }
          ],
          "responses": {
            "400": {
              "description": "Invalid user supplied"
            },
            "404": {
              "description": "User not found"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        },
        "delete": {
          "tags": [
            "user"
          ],
          "summary": "Delete user",
          "description": "This can only be done by the logged in user.",
          "operationId": "deleteUser",
          "produces": [
            "application/xml",
            "application/json"
          ],
          "parameters": [
            {
              "name": "username",
              "in": "path",
              "description": "The name that needs to be deleted",
              "required": true,
              "type": "string"
            }
          ],
          "responses": {
            "400": {
              "description": "Invalid username supplied"
            },
            "404": {
              "description": "User not found"
            }
          },
          "x-accepts": "application/json",
          "x-contentType": "application/json"
        }
      }
    },
    "host": "petstore.swagger.io",
    "schemes": [
      "http"
    ],
    "externalDocs": {
      "description": "Find out more about Swagger",
      "url": "http://swagger.io"
    },
    "securityDefinitions": {
      "petstore_auth": {
        "type": "oauth2",
        "authorizationUrl": "http://petstore.swagger.io/oauth/dialog",
        "flow": "implicit",
        "scopes": {
          "write:pets": "modify pets in your account",
          "read:pets": "read your pets"
        }
      },
      "api_key": {
        "type": "apiKey",
        "name": "api_key",
        "in": "header"
      }
    },
    "definitions": {
      "Order": {
        "type": "object",
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64"
          },
          "petId": {
            "type": "integer",
            "format": "int64"
          },
          "quantity": {
            "type": "integer",
            "format": "int32"
          },
          "shipDate": {
            "type": "string",
            "format": "date-time"
          },
          "status": {
            "type": "string",
            "description": "Order Status",
            "enum": [
              "placed",
              "approved",
              "delivered"
            ]
          },
          "complete": {
            "type": "boolean",
            "default": false
          }
        },
        "xml": {
          "name": "Order"
        }
      },
      "Category": {
        "type": "object",
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64"
          },
          "name": {
            "type": "string"
          }
        },
        "xml": {
          "name": "Category"
        }
      },
      "User": {
        "type": "object",
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64"
          },
          "username": {
            "type": "string"
          },
          "firstName": {
            "type": "string"
          },
          "lastName": {
            "type": "string"
          },
          "email": {
            "type": "string"
          },
          "password": {
            "type": "string"
          },
          "phone": {
            "type": "string"
          },
          "userStatus": {
            "type": "integer",
            "format": "int32",
            "description": "User Status"
          }
        },
        "xml": {
          "name": "User"
        }
      },
      "Tag": {
        "type": "object",
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64"
          },
          "name": {
            "type": "string"
          }
        },
        "xml": {
          "name": "Tag"
        }
      },
      "ApiResponse": {
        "type": "object",
        "properties": {
          "code": {
            "type": "integer",
            "format": "int32"
          },
          "type": {
            "type": "string"
          },
          "message": {
            "type": "string"
          }
        }
      },
      "Pet": {
        "type": "object",
        "required": [
          "name",
          "photoUrls"
        ],
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64"
          },
          "category": {
            "$ref": "#/definitions/Category"
          },
          "name": {
            "type": "string",
            "example": "doggie"
          },
          "photoUrls": {
            "type": "array",
            "xml": {
              "name": "photoUrl",
              "wrapped": true
            },
            "items": {
              "type": "string"
            }
          },
          "tags": {
            "type": "array",
            "xml": {
              "name": "tag",
              "wrapped": true
            },
            "items": {
              "$ref": "#/definitions/Tag"
            }
          },
          "status": {
            "type": "string",
            "description": "pet status in the store",
            "enum": [
              "available",
              "pending",
              "sold"
            ]
          }
        },
        "xml": {
          "name": "Pet"
        }
      }
    },
    "swagger": "2.0",
    "info": {
      "description": "This is a sample server Petstore server.  You can find out more about Swagger at [http://swagger.io](http://swagger.io) or on [irc.freenode.net, #swagger](http://swagger.io/irc/).  For this sample, you can use the api key `special-key` to test the authorization filters.",
      "version": "1.0.0",
      "title": "Swagger Petstore",
      "termsOfService": "http://swagger.io/terms/",
      "contact": {
        "email": "apiteam@swagger.io"
      },
      "license": {
        "name": "Apache 2.0",
        "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
      }
    },
    "tags": [
      {
        "name": "pet",
        "description": "Everything about your Pets",
        "externalDocs": {
          "description": "Find out more",
          "url": "http://swagger.io"
        }
      },
      {
        "name": "store",
        "description": "Access to Petstore orders"
      },
      {
        "name": "user",
        "description": "Operations about user",
        "externalDocs": {
          "description": "Find out more about our store",
          "url": "http://swagger.io"
        }
      }
    ]
  },
  "component": [
    {
      "moduleName": "com.networknt.info.ServerInfoHandler",
      "config": {
        "description": "Server info endpoint that can output environment and component along with configuration",
        "enableServerInfo": true
      }
    },
    {
      "moduleName": "com.networknt.validator.ValidatorHandler",
      "config": {
        "description": "Validate request/response against swagger spec during runtime",
        "enableValidator": true,
        "enableResponseValidator": false
      }
    },
    {
      "moduleName": "com.networknt.info.SimpleAuditHandler",
      "config": {
        "description": "controls how audit info should be logged. FullAudit is not recommended on produciton if performance is important",
        "enableFullAudit": false,
        "simple": {
          "statusCode": true,
          "responseTime": true,
          "headers": [
            "correlationId",
            "traceabilityId",
            "clientId",
            "userId",
            "scopeClientId",
            "endpoint"
          ]
        },
        "enableSimpleAudit": true,
        "full": {
          "enableMask": true,
          "request": {
            "headers": [
              "contentType",
              "characterEncoding"
            ],
            "cookies": true,
            "queryParameters": true,
            "body": true
          },
          "response": {
            "headers": true,
            "cookies": true,
            "body": true,
            "statusCode": true,
            "contentLength": true
          }
        }
      }
    }
  ]
}
```
