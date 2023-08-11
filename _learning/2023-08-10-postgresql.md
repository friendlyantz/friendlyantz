---
# layout: posts
# author_profile: true
title: "PostreSQL"
permalink: /postgres/
excerpt: "my findings and tricks"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - databases
  - postgres
# toc: true
# toc_sticky: true
# toc_label: "My Table of Contents"
# toc_icon: "cog"
---

> Pay Attention: command execution happens only after `;`-char. 'Enter' only helps with formatting on a new line

# Creation

```
psql
CREATE DATABASE sandbox;

psql sandbox
CREATE TABLE user (
    Column name + data type + constraints
    id BIGSERIAL NOT NULL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    gender VARCHAR(6) NOT NULL,
    fav_num int,
    dob TIMESTAMP / DATE NOT NULL
);
```
[data_types in postgres](https://www.postgresql.org/docs/current/datatype.html)


```sh
# do not do this
DROP DATABASE sandbox;

# connect to diff DATABASE
\c name_of_other_db

\d - describe, or \dt - w/o index / sequence
\d table_name - show column_types

```

# Insertion

> USE `'sincle quotes'`
> Note date format: YYYY-MM-DD

```sh
INSERT INTO person (name, fav_num, dob)
VALUES ('Antz', 9, date '1987-04-01');

# alternatively use a '.sql' file via psql \i command
psql
\i  /Users/somepath.sql
```
