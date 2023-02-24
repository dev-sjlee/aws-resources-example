---
title: SQL Example Queries
description: Example SQL queries for MySQL, PostgreSQL.
---

# Example Queries

## About Connection

### Connect to Database

=== "MySQL"
    ``` sql hl_lines="1"
    mysql -u USERNAME -h HOSTNAME -P PORT -p
    ```

=== "PostgreSQL"
    ``` sql hl_lines="1"
    psql -U USERNAME -h HOSTNAME -p PORT
    ```

## About Database

### Create Database

=== "MySQL"
    ``` sql hl_lines="1"
    CREATE DATABASE `DB_NAME`;
    ```

=== "PostgreSQL"
    ``` sql hl_lines="1"
    CREATE SCHEMA "DB_NAME";
    ```

### Use Database

=== "MySQL"
    ``` sql hl_lines="1"
    USE `DB_NAME`;
    ```

=== "PostgreSQL"
    ``` sql hl_lines="1"
    \c "DB_NAME"
    ```

### List Databases

=== "MySQL"
    ``` sql
    SHOW DATABASES;
    ```

=== "PostgreSQL"
    ``` sql
    \dn
    ```

### Delete Database

=== "MySQL"
    ``` sql hl_lines="1"
    DROP DATABASE `DB_NAME`;
    ```

=== "PostgreSQL"
    ``` sql hl_lines="1"
    DROP SCHEMA "DB_NAME";
    ```

## About Table

### Create Table

=== "MySQL"
    ``` sql hl_lines="1"
    CREATE TABLE `DB_NAME`.`TABLE_NAME` (
        IDX INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        NAME VARCHAR(255),
        OTHER_ID INT,
        INDEX NAME_IDX(NAME),
        FOREIGN KEY (OTHER_ID) REFERENCES `OTHER_TABLE_NAME` (OTHER_TABLE_COLUMN_NAME) ON UPDATE CASCADE
    );
    ```

=== "PostgreSQL"
    ``` sql hl_lines="1"
    CREATE TABLE "DB_NAME"."TABLE_NAME" (
        IDX INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        NAME VARCHAR(255),
        OTHER_ID INT,
        FOREIGN KEY (OTHER_ID) REFERENCES "DB_NAME"."OTHER_TABLE_NAME" (OTHER_TABLE_COLUMN_NAME) ON UPDATE CASCADE
    );

    CREATE INDEX "INDEX_NAME" ON "DB_NAME"."TABLE_NAME" (COLUMN_NAME);
    ```