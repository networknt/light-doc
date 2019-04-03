---
title: "API Document"
date: 2018-03-22T22:03:16-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

### Format Schemes

Token schemes need to be flexible enough to handle multiple formats for the sensitive data they accept — such as personally identifiable information, Social Insurance Numbers, and credit card numbers. In some cases additional format constraints must be honored. As an example, a token representing a Social Insurance Number in a customer service application may need to retain the real last 4 digits. This enables customer service representatives to verify user identities without access to the rest of the SIN.

When tokenizing credit cards, tokens are the same size as the original credit card number and pass the LUHN check. As the token still resembles a real card number, systems that use the numbers need not be altered to accommodate tokens. However, unlike the real credit card or Social Insurance Numbers, tokens cannot be used as financial  instruments, and have no value other than as a reference to the original transaction or real account. The relationship between a token and a card number is unique for any given payment system, so even if someone compromises the entire token database that they can commit transactions in that system (a rare but real possibility), the numbers are worthless outside the environment they were created for. And most important, real tokens cannot be decrypted or otherwise restored back to the original credit card number.

Each data type has different use cases and tokenization service needs to offer various generation options to accommodate them. There are nine schemes supported out of the box, and they cover a majority of use cases for enterprise-scale tokenization requirement. The following is the database script with descriptions. 


```sql
INSERT INTO format_scheme (id, name, description)
VALUES (0, 'UUID', 'UUID Version 4 Universally Unique Identifier. This format requires no validation and will return a UUID as the token.');
INSERT INTO format_scheme (id, name, description)
VALUES (1, 'GUID', 'Globally Unique Identifier. This format requires no validation and will return a GUID as the token.');
INSERT INTO format_scheme (id, name, description)
VALUES (2, 'LN', 'LUHN Compliant Numeric. This format is used to tokenize a social insurance number or social security number.');
INSERT INTO format_scheme (id, name, description)
VALUES (3, 'N', 'Random Numeric. This format is used to tokenize a account number or any number without validation.');
INSERT INTO format_scheme (id, name, description)
VALUES (4, 'LN4', 'LUHN Compliant Numeric token retaining the original last 4 digits of the number. Can be used as credit card number.');
INSERT INTO format_scheme (id, name, description)
VALUES (5, 'AN', 'Alpha Numeric, length preserving token.');
INSERT INTO format_scheme (id, name, description)
VALUES (6, 'AN4', 'Alpha Numeric, length preserving token retaining the original last 4 characters.');
INSERT INTO format_scheme (id, name, description)
VALUES (7, 'CC', 'Credit Card Number, LUHN length preserving token retaining the original first character.');
INSERT INTO format_scheme (id, name, description)
VALUES (8, 'CC4', 'Credit Card Number, LUHN length preserving token retaining the original first digit and the last 4 digits.');

```

To get the above info from scheme endpoint: 

```
curl -k -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzEwODM3NCwianRpIjoiNUVIWDFzXzlDZkhXQjY3dk1XOHRudyIsImlhdCI6MTUyMTc0ODM3NCwibmJmIjoxNTIxNzQ4MjU0LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyIsInNjaGVtZS5yIl19.r_3u4HAJpCztcX8HhV5kihSj6V2gBbqfB4Bdjr3arRKHKJdncaaoDRcYgXihdtutBsA7QVRimu576HL6FwV9iurpqEEA-uy-rzfuCfXJYP4s4F5C_PFaeroGi9siG_dc34p-iFh6eA6dksa6pwBiko9Pb1eBO8XfIV7ndNUmqTUbEvjV6J_Nv_aVPoDgOz00laMDDgj3bOtkz3lGTrfZCQloAhagthfMcUzyj04qe_bKZFKcrbCxXfBjelItUBwdt1td8FBpiSQPI0FXVud69TFmDmDZT6UXci8qJVOb0vuADJcPFe5PEWXxIORoduU8an0Mtey5svQx3c0W_Gqvcg' https://198.55.49.187:6888/v1/scheme
```

Result: 

```json
[
  {
    "name": "UUID",
    "description": "UUID Version 4 Universally Unique Identifier. This format requires no validation and will return a UUID as the token.",
    "id": "0"
  },
  {
    "name": "GUID",
    "description": "Globally Unique Identifier. This format requires no validation and will return a GUID as the token.",
    "id": "1"
  },
  {
    "name": "LN",
    "description": "LUHN Compliant Numeric. This format is used to tokenize a social insurance number or social security number.",
    "id": "2"
  },
  {
    "name": "N",
    "description": "Random Numeric. This format is used to tokenize a account number or any number without validation.",
    "id": "3"
  },
  {
    "name": "LN4",
    "description": "LUHN Compliant Numeric token retaining the original last 4 digits of the number. Can be used as credit card number.",
    "id": "4"
  },
  {
    "name": "AN",
    "description": "Alpha Numeric, length preserving token.",
    "id": "5"
  },
  {
    "name": "AN4",
    "description": "Alpha Numeric, length preserving token retaining the original last 4 characters.",
    "id": "6"
  },
  {
    "name": "CC",
    "description": "Credit Card Number, LUHN length preserving token retaining the original first character.",
    "id": "7"
  },
  {
    "name": "CC4",
    "description": "Credit Card Number, LUHN length preserving token retaining the original first digit and the last 4 digits.",
    "id": "8"
  }
]

```

There is also an endpoint that you can get a particular scheme format info: 

```
curl -k -H 'Authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzEwODM3NCwianRpIjoiNUVIWDFzXzlDZkhXQjY3dk1XOHRudyIsImlhdCI6MTUyMTc0ODM3NCwibmJmIjoxNTIxNzQ4MjU0LCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyIsInNjaGVtZS5yIl19.r_3u4HAJpCztcX8HhV5kihSj6V2gBbqfB4Bdjr3arRKHKJdncaaoDRcYgXihdtutBsA7QVRimu576HL6FwV9iurpqEEA-uy-rzfuCfXJYP4s4F5C_PFaeroGi9siG_dc34p-iFh6eA6dksa6pwBiko9Pb1eBO8XfIV7ndNUmqTUbEvjV6J_Nv_aVPoDgOz00laMDDgj3bOtkz3lGTrfZCQloAhagthfMcUzyj04qe_bKZFKcrbCxXfBjelItUBwdt1td8FBpiSQPI0FXVud69TFmDmDZT6UXci8qJVOb0vuADJcPFe5PEWXxIORoduU8an0Mtey5svQx3c0W_Gqvcg' https://198.55.49.187:6888/v1/scheme/1
```

Result: 

```json
{
  "name": "GUID",
  "description": "Globally Unique Identifier. This format requires no validation and will return a GUID as the token.",
  "id": "1"
}

```

### Create a token

The body of the enpoint looks something like this: 

```json
{
  "schemeId": 1,
  "value": "1234567890"
}
```

The schemeId is one of the scheme format id in above endpoint. Pick the right format based on your use case. 


```
curl -k -X POST https://198.55.49.187:6888/v1/token -H 'content-type: application/json' -H 'authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg' -d '{"schemeId":1,"value":"1234567890"}'
```

Result:

```
CB_ATIeHSnu5dTv40Vz2-Q
```

### Detokenize

```
curl -k -H 'authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg' https://198.55.49.187:6888/v1/token/CB_ATIeHSnu5dTv40Vz2-Q
```

Result: 

```
1234567890
```

### Delete a Token

```
curl -k -X DELETE https://198.55.49.187:6888/v1/token/CB_ATIeHSnu5dTv40Vz2-Q -H 'authorization: Bearer eyJraWQiOiIxMDAiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJ1cm46Y29tOm5ldHdvcmtudDpvYXV0aDI6djEiLCJhdWQiOiJ1cm46Y29tLm5ldHdvcmtudCIsImV4cCI6MTgzNzA5NDMzMywianRpIjoiUk1RZXN0MUVBTGY5ZFJHSl8tSEowQSIsImlhdCI6MTUyMTczNDMzMywibmJmIjoxNTIxNzM0MjEzLCJ2ZXJzaW9uIjoiMS4wIiwidXNlcl9pZCI6InN0ZXZlIiwidXNlcl90eXBlIjoiRU1QTE9ZRUUiLCJjbGllbnRfaWQiOiJmN2Q0MjM0OC1jNjQ3LTRlZmItYTUyZC00YzU3ODc0MjFlNzIiLCJzY29wZSI6WyJ0b2tlbi5yIiwidG9rZW4udyJdfQ.fVmHPr5vlDf01zhIiRio1N4-lRIaKShjWis1lEJOXx3oHnkqWWiog0JWIw7R_7b5siPgvPuJOMSi5zDQjva9D-EiIRGGcwp9Egb_gqOLLvMaux3fZrzYX8WSk_VcpDtbHb303DyAWuOkgMM9VamDZT6sP66qTAVU5Ao0iS9bi3kTyv13_To2nXVDeb1FTTXHcw8gSY-2HpjsIx5IDf8rayMMp1p1Y6heyQBrVN5qJd1UhmWwuzsj3VwX_iSx-qw7AFZResTobHntRlbPX5D2Xo0fMDZ7HR8JzT_32aWLheOmionfOFUeuve9WtDk5c0TypcNMgiJi6WVjYcjZCcmBg' 
```

There is nothing to return for this endpoint.


