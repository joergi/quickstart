---
title: "Exclude one element from Lombok model-wide setter and getter"
description: "Using Lombok @Setter and @Getter for all elements, but skip it for one element"
categories: [java, lombok]
tags: [lombok, java]
date: 2020-06-07
---

Lets say you have a User model with [Lombok](https://projectlombok.org/) modelwide [@Getter](https://projectlombok.org/api/lombok/Getter.html) and [@Setter](https://projectlombok.org/api/lombok/Setter.html):

```java
@Getter
@Setter
public class User {

    @Id
    private String id;

    private String username;

    private String email;

    private String password;
}
```

```setId(String id)``` makes no sense.    
Normally the id is set by the DB, so it makes no sense to set it.

So you can use ```  @Setter(AccessLevel.NONE)```   (or   ```@Getter(AccessLevel.NONE)```) to exclude it from the Lombok setters (or getters)

```java
@Getter
@Setter
public class User {

    @Id
    @Setter(AccessLevel.NONE)
    private String id;

    private String username;

    private String email;

    private String password;
}
```

(I found this answer on [Stackoverflow](https://stackoverflow.com/a/7994134/863403) by user [Michael Piefel](https://stackoverflow.com/users/2621917/michael-piefel) for this problem, thanks!
