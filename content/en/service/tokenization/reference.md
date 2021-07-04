---
title: "Tokenization Reference"
date: 2018-03-25T10:28:00-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

While implementing our tokenization service, we have evaluated several commercial products and open source offerings. Our philosophy is to pick the right open source product that meets our requirement and can be easily integrated into our ecosystem. When there is no open source project suitable for our needs, then we consider commercial offerings. Our last resort is to implement it on our own. 

### Open Source

There is an open source project [Tokenator][] available on GitHub.com, but it is very simple and hasn't been updated the last three years. 

### Commercial

* [tokenex] is a cloud service provider but doesn't support OAuth 2.0 yet. 
* [emvco] is another online service provider but focuses on payment only.

### Tools

There are some useful online tools we have used during the development: 

* [Credit Card Number Validator][]

### Other Materials

https://www.pcisecuritystandards.org/documents/Tokenization_Product_Security_Guidelines.pdf
https://securosis.com/assets/library/reports/Securosis_Understanding_Tokenization_V.1_.0_.pdf
http://apps.cybersource.com/library/documentation/dev_guides/tokenization_SO_API/Tokenization_SO_API.pdf

http://blog.cleverelephant.ca/2014/06/tokenization-and-your-private-data-1.html
http://blog.cleverelephant.ca/2014/07/tokenization-and-your-private-data-2.html
http://blog.cleverelephant.ca/2014/07/tokenization-and-your-private-data-3.html
http://blog.cleverelephant.ca/2014/07/tokenization-and-your-private-data-4.html
http://blog.cleverelephant.ca/2014/07/tokenization-and-your-private-data-5.html

https://medium.com/@blake_hall/moving-beyond-social-security-numbers-part-3-283bbf28ce74

https://docs.tokenex.com/?page=appendix#token-schemes
https://developer.cardconnect.com/cardsecure-api#api-action-codes

[Tokenator]: https://github.com/SimplyTapp/Tokenator
[Credit Card Number Validator]: https://www.datageneratortools.com/card/validator
[tokenex]: https://tokenex.com/
[emvco]: https://www.emvco.com/emv-technologies/payment-tokenisation/
