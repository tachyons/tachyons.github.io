---
layout: single
title: 'Intermediate Postgresql for rails developers, Part 0: Get your environment'
description: 'Part 0 for Intermediate postgresql guide'
tags: ['psql', 'rails', 'postresql']
date: 2021-04-30 23:12 +0530
---

Postgresql is one of the most advanced open source database available in the market. Adherence to the SQL standards and super cool extra features are few of the reason by Postgres being the most popular database among rails community. This is neither introduction to Postgres nor rails. This series is for the folks who have been using rails and Postgres for one or more years. 

In part 0 we will walk through the tips for postgres setup to increase the productivity.

ActiveRecord orm which is built into rails provides a lot of cool methods to access the data in rubyistic way. But it is important to get familiarize with the native Postgres tooling for many tasks.

## Psql

  Psql is the official commandline client for postgresql, rails does have shortcut to open `psql` using the configuration from database configuration as `rails dbconsole`. But you can also open by using `psql` command on your command line client.
  
  `psql -h localhost -U username databasename`

  Above is the basic syntax for opening psql console, Once you enter that you will be greeted with psql console if the credentials given are correct.

  Now let's go through few psql tips

### Backward slash commands

`psql` supports a lot of useful configuration options to improve the experiance in postgresql shell.

* `\timing`

  Timing command add the time taken to run the command, this is handy when optimizing query performance

```sql
[local] abomk@cv_dump=#SELECT COUNT(id) from users;
? 45129 ?
?????????

[local] abomk@cv_dump=#\timing
Timing is on.
[local] abomk@cv_dump=#SELECT COUNT(id) from users;
? 45129 ?
?????????

Time: 8.412 ms
```
* `\s`

  Get posgtres commands history, this is handy for documenting the stuff after doing experiments with different queries and many more. You can also choose to save the history by providing filename as the parameter as `\s filename`

* `\i filename`

Load query from file. Big queries are often convent to write in a text editor, `\i` enables loading a query from a file and execute it.

* `\e`

This another approach to solve the difficulty of writing multi line queries in a shell environment, When you give `\e` command, psql will open the text editor you set $EDITOR to open the query. You can edit and close the file, content will be copied to posql shell, and you can execute.


You can find more such commands from [posgrestutorials](https://www.postgresqltutorial.com/psql-commands/) and [pgdash](https://pgdash.io/blog/postgres-psql-tips-tricks.html)

### Managing multiple versions of postgres

It is important to have the same version of postgres in your development setup as the production version. If you have multiple apps with diffrent versions of postgres versions, you can use [pgenv](https://github.com/theory/pgenv) to configure multiple versions of Postgres in your local. There are other options like [asdf-postgres](https://github.com/smashedtoatoms/asdf-postgres)

### Use different pager

The default pager is somewhat difficult to read when there are too many columns. [pspg](https://github.com/okbob/pspg) is an alternate pager you can use with Postgres


### Save your psql settings in your psqlrc

You can save the psql configurations we discussed above in a file so that you don't need to repeat every time you open the psql shell. Here is my psql configuration for reference


```SQL
\set ON_ERROR_ROLLBACK interactive
\set COMP_KEYWORD_CASE upper
\set HISTFILE ~/.psql/history- :DBNAME
\set VERBOSITY verbose
\set PROMPT1 '%[%033[1m%]%M %n@%/%R%[%033[0m%]%#'
\setenv PAGER pspg
\pset border 2
\pset linestyle unicode
\set null '(null)'
```
You can find explanation of individual configurations from [thoughtbot blog](https://thoughtbot.com/blog/an-explained-psqlrc)

## Pgcli

[Pgcli](https://www.pgcli.com/) is an alternate postgresql client with additional features like autocompletion of table and sql queries. 

We will go through more tips in coming articles, while it is not mandatory to follow the above setup for cominig articles, it is good to have to get hands on with psql shell


