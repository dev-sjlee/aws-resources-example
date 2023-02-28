---
title: Build with Cache
description: Cache settings for CodeBuild.
---

# Build with Cache

## Go

```
/go/pkg/mod/**/*
```

## NodeJS

```
/root/.npm/**/*
/usr/lib/node_modules/**/*
```

## PHP

### Composer

```
/root/.composer/**/*
```

## Java

### Gradle

```
/root/.gradle/**/*
```

### Maven

```
/root/.m2/**/*
```

## Python

!!! Note

    You should fix Python version.

```
/usr/local/lib/pythonO.O/site-package/
```