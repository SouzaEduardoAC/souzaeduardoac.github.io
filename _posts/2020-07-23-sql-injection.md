---
layout: post
title: "SQL Injection"
summary: "How does it work and how can we prevent it"
author: souzaeduardoac
date: '2020-07-23 17:21:23 -0300'
category: security
thumbnail: /assets/img/posts/sql_injection.jpg
permalink: /blog/sql-injection/
---

**SQL Injection** is a code injection technique, used to attack applications, in which malicious SQL statements are inserted in client (web/mobile) fields to be executed by the database and retrieve, update or even wipe data.

Prepared statements (aka parameterized queries) are the best mechanism for preventing SQL injection attacks. Used to abstract SQL statement syntax from input parameters, statement templates are first defined at the application layer, and the parameters are then passed to them.

Aside from a better protection against SQL injection attacks, prepared statements offer improved code quality from a legibility and maintainability perspective due to the separation of the SQL logic from its inputs.

**Second-Order SQL injection** is a less visible kind of good old SQL injection. Unlike in regular SQL injection, user input doesn't influence the SQL query directly. Instead, a malicious payload from the user input is, first, saved to the database, and only after that, it is used by the application in the SQL query.

The typical flow of the Second-Order SQL Injection:
 * Tainted data is inserted into the database securely using an approach that prevents SQL injections.
 * Tainted data is then insecurely used as a part of an SQL query leading to the SQL injection. 

Prepared statements (aka parameterized queries) are an effective way to prevent all kinds of SQL injection attacks. Used to abstract SQL statement syntax from input parameters. Statement templates are first defined at the application layer, and the parameters are then passed to them.  

Another way to avoid SQL injections is to use ORM. It provides a developer with a way to manipulate the database without writing SQL queries by setting relations between database tables and application objects.

> ORM allows usage of raw SQL queries that are vulnerable to SQL injection if the query is built by concatenating user input with the SQL keywords. Do not concatenate user-supplied data with SQL queries performed by ORM.

Do not banlist or encode potentially dangerous characters like single or double-quotes. This approach is flawed because a banlist can eventually be bypassed. 

All SQL queries must be performed in a secure way regardless of whether or not you're planning to utilize user input in the query. The data stored in the database should always be treated as untrusted, and defense-in-depth approach should be applied. 