---
title: "SPA CSRF Prevention"
date: 2018-05-22T12:48:01-04:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

Cross-Site Request Forgery (CSRF) is a type of attack that occurs when a malicious web site, email, blog, instant message, or program causes a user’s web browser to perform an unwanted action on a trusted site for which the user is currently authenticated. The impact of a successful CSRF attack is limited to the capabilities exposed by the vulnerable application. For example, this attack could result in a transfer of funds, changing a password, or purchasing an item in the user's context. In effect, CSRF attacks are used by an attacker to make a target system perform a function via the target's browser without knowledge of the target user, at least until the unauthorized transaction has been committed.

Impacts of successful CSRF exploits vary greatly based on the privileges of each victim. When targeting a normal user, a successful CSRF attack can compromise end-user data and their associated functions. If the targeted end user is an administrator account, a CSRF attack can compromise the entire web application. Sites that are more likely to be attacked by CSRF are community websites (social networking, email) or sites that have high dollar value accounts associated with them (banks, stock brokerages, bill pay services). Utilizing social engineering, an attacker can embed malicious HTML or JavaScript code into an email or website to request a specific 'task URL'. The task then executes with or without the user's knowledge, either directly or by utilizing a Cross-Site Scripting flaw (ex: Samy MySpace Worm).