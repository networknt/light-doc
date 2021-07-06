---
title: "Data Structure Change"
date: 2020-03-25T15:12:50-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
reviewed: true
---

While working on the light-portal, I sometimes needed to change the data structure or schema for an endpoint. As the portal application has too many moving parts, I am using this document to collect all the necessary changes in each repository and project as a reminder. It can be a guide for new developers in the future. 

The change for today is to remove the email from the change password endpoint. We don't need to have an additional email field in the form because only a logged-in user can take action. The email can be retrieved from the JWT token on the server.

### model-config

Let's make the change from the service schema first. The user-command service [schema](https://github.com/networknt/model-config/blob/master/hybrid/user-command/schema.json) is located in the model-config for the project generation. 

```
steve@freedom:~/networknt/model-config$ git diff hybrid/user-command/schema.json
diff --git a/hybrid/user-command/schema.json b/hybrid/user-command/schema.json
index 898a1dc..3984f87 100644
--- a/hybrid/user-command/schema.json
+++ b/hybrid/user-command/schema.json
@@ -205,10 +205,6 @@
         "title": "Service",
         "type": "object",
         "properties": {
-          "email": {
-            "type": "string",
-            "description": "email address of the user"
-          },
           "oldPassword": {
             "type": "string",
             "description": "the old password"
@@ -223,7 +219,6 @@
           }
         },
         "required": [
-          "email",
           "oldPassword",
           "newPassword",
           "passwordConfirm"

```

In the JSON schema, we remove the email field and email from the required. 

Here is the [issue](https://github.com/networknt/model-config/issues/65) that is tracking the change.

### portal-event

This is a component that contains all the Avro schemas and generated Java code. The user-command service will convert the input into an event in Avro format and send it to Kafka so that user-query can get the streams to process it. The change is something like below. 

```
steve@freedom:~/networknt/light-portal$ git diff portal-event/src/main/avro/PasswordChangedEvent.avsc
diff --git a/portal-event/src/main/avro/PasswordChangedEvent.avsc b/portal-event/src/main/avro/PasswordChangedEvent.avsc
index f1096c0..bef1a21 100644
--- a/portal-event/src/main/avro/PasswordChangedEvent.avsc
+++ b/portal-event/src/main/avro/PasswordChangedEvent.avsc
@@ -22,11 +22,6 @@
           }
         ]
       }
-    },
-     {
-      "name": "email",
-      "type": "string",
-      "doc": "email of the user to be deleted"
     },
     {
       "name": "password",

```

After the Avro schema file is changed, we need to compile it to generate the Java code. Please use the following commands to do that. 

```
cd ~/networknt/light-portal/portal-event/src/main/avro
./compile.sh
```

To compile the generated code, we run the maven command. 

```
cd ~/networknt/light-portal/portal-event
mvn clean install
```

Here is an [issue](https://github.com/networknt/light-portal/issues/142) that is tracking the change. 


### user-command

The service user-command is the one to handle the changePassword endpoint. It was generated from the schema in the model-config above. To make it simple, we are not going to regenerate the project again, just make the change in the schema file like below. 


```
steve@freedom:~/networknt/light-portal/user-command$ git diff src/main/resources/schema.json
diff --git a/user-command/src/main/resources/schema.json b/user-command/src/main/resources/schema.json
index 5d5a912..592dd7f 100644
--- a/user-command/src/main/resources/schema.json
+++ b/user-command/src/main/resources/schema.json
@@ -19,10 +19,6 @@
       "title": "Service",
       "type": "object",
       "properties": {
-        "email": {
-          "type": "string",
-          "pattern": "^\\S+@\\S+$"
-        },
         "oldPassword": {
           "type": "string",
           "description": "the old password"
@@ -37,7 +33,6 @@
         }
       },
       "required": [
-        "email",
         "oldPassword",
         "newPassword",
         "passwordConfirm"

```

Here is the [issue](https://github.com/networknt/light-portal/issues/141) that is tracking the change.

Of course, we need to change the ChangePassword.java in the handler folder to remove the email from the input. The detailed change in Java code is combined with others and can be viewed in this [issue](https://github.com/networknt/light-portal/issues/143).

### user-query

The user-query service has a stream processor that handles all the events. We need to double-check the PasswordChangedEvent logic. Here is the changes we have made to the UserQueryStreams.java

```
steve@freedom:~/networknt/light-portal/user-query$ git diff src/main/java/net/lightapi/portal/user/query/UserQueryStreams.java
diff --git a/user-query/src/main/java/net/lightapi/portal/user/query/UserQueryStreams.java b/user-query/src/main/java/net/lightapi/portal/user/query/UserQueryStreams.java
index 5218d3a..b870a50 100644
--- a/user-query/src/main/java/net/lightapi/portal/user/query/UserQueryStreams.java
+++ b/user-query/src/main/java/net/lightapi/portal/user/query/UserQueryStreams.java
@@ -306,9 +306,8 @@ public class UserQueryStreams implements LightStreams {
                 // make sure that we have user available.
                 PasswordChangedEvent passwordChangedEvent = (PasswordChangedEvent)object;
                 if(logger.isTraceEnabled()) logger.trace("Event = " + passwordChangedEvent);
-                String id = passwordChangedEvent.getEventId().getId();
+                String email = passwordChangedEvent.getEventId().getId();
                 long nonce = passwordChangedEvent.getEventId().getNonce();
-                String email = passwordChangedEvent.getEmail();
                 String password = passwordChangedEvent.getPassword();
                 String oldPassword = passwordChangedEvent.getOldPassword();
                 String storedPassword = passwordStore.get(email);
@@ -326,7 +325,7 @@ public class UserQueryStreams implements LightStreams {
                     addNotification(email, new EventNotification(nonce, false, "Failed to validate the old password.", AvroConverter.toJson(passwordChangedEvent, false)));
                     return;
                 }
-                nonceStore.put(id, nonce + 1);
+                nonceStore.put(email, nonce + 1);
                 addNotification(email, new EventNotification(nonce, true, null, AvroConverter.toJson(passwordChangedEvent, false)));
             } else if(object instanceof UserDeletedEvent) {
                 // make sure that we have user available.

```

Here is the [issue](https://github.com/networknt/light-portal/issues/144) that is tracking the change.

### view

The last step is to update the form definition in the portal view project for the UI. We don't have the form defined yet, so we add a new form in the Form.json file.

```
steve@freedom:~/networknt/light-portal$ git diff view/src/data/Forms.json
diff --git a/view/src/data/Forms.json b/view/src/data/Forms.json
index dd8a479..9765c6b 100644
--- a/view/src/data/Forms.json
+++ b/view/src/data/Forms.json
@@ -167,6 +167,52 @@
       "address"
     ]
   },
+  "changePasswordForm": {
+    "formId": "changePasswordForm",
+    "actions": [
+      {
+        "host": "lightapi.net",
+        "title": "Change Password",
+        "success": "/#/app/success",
+        "failure": "/#/app/failure"
+      }
+    ],
+    "schema": {
+      "title": "Change Password",
+      "type": "object",
+      "properties": {
+        "oldPassword": {
+          "type": "string"
+        },
+        "newPassword": {
+          "type": "string"
+        },
+        "passwordConfirm": {
+          "type": "string"
+        }
+      },  
+      "required": [
+        "oldPassword",
+        "newPassword",
+        "passwordConfirm"
+      ]
+    },
+    "form": [
+      {
+        "key": "oldPassword",
+        "type": "password"
+      },
+      {
+        "key": "newPassword",
+        "type": "password"
+      },
+      {
+        "key": "passwordConfirm",
+        "type": "password"
+      }
+    ]
+  },
+
   "service": {
     "schema": {
       "type": "object",

```

Here is the [issue](https://github.com/networknt/light-portal/issues/145) that is tracking the change.


### Summary

As you can see, changing the data structure in the light-portal is not an easy job. There are so many places that need to be touched; however, the changes are relatively easy to understand. 

If you need another example, please take a look at this [issue](https://github.com/networknt/model-config/issues/67). 

To ease the pain to change the JSON schema in model-config, user-command, and view, we are going to develop a JSON schema market place in the light-portal so that we can only change one location and all three are using a remote reference. 




