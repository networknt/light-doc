---
title: "Utility"
date: 2017-11-05T10:24:06-05:00
description: ""
categories: [concerns]
keywords: []
aliases: []
toc: false
draft: false
reviewed: true
---


This module contains some useful classes that are shared by multiple modules within the light-* frameworks.

# Constants

Contains all the constants shared by all modules.

# ModuleRegistry

When the plugin modules are loaded, it will register itself to this module along
with configuration. When /server/info is called, the endpoint will return all
plugged in modules and their configurations.

# Util

Some useful utility method like uuid generator etc.

# CollectionUtil

Utility for collection

# ConcurrentHashSet

ConcurrentHashSet implementation

# HashUtil

Hash calculation utility

# NetUtils

Utilities for network

# NioUtils

NIO utility



