---
title: "Kafkacat Command Line Tool"
date: 2020-05-08T22:37:08-04:00
description: ""
categories: []
keywords: []
slug: ""
toc: false
draft: false
---

### Install

sudo apt install kafkacat

### Usage

```
kafkacat -b localhost -t portal-event

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t covid-query-covid-entity-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t covid-query-covid-status-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t covid-query-covid-website-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t nonce-query-user-nonce-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t notification-query-user-notification-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-city
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-event
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-map
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-nonce
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-notification
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-taiji
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-userid
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-reference
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-password-token-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-user-email-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-user-message-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-user-password-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-user-token-store-changelog

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-user-payment-store-changelog

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-customer-order-store-changelog

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t user-query-merchant-order-store-changelog

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t portal-host

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t market-query-market-client-store-changelog

kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t market-query-market-host-store-changelog
kafkacat -b localhost -f '\nKey (%K bytes): %k\t\nValue (%S bytes): %s\n\Partition: %p\tOffset: %o\n--\n' -t market-query-market-token-store-changelog


```
