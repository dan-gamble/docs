---
title: Lucid getters and setters
description: getters,setters and computed properties in lucid ORM.
permalink: lucid-getters-setters
weight: 1
categories:
    - Database
versions:
    - 3.0
---

# Getters/Setters

This guide will explain how to mutate and access data from your model instances without repeating yourself. By the end of this guide, you will know.

1. You do define getters/setters for a model.
2. How to create computed properties.
3. What role getters/setters can play for your application.

@partial(data-properties)

## Introduction

One of the important steps towards building data driven applications is to control the flow of data. That what data properties helps you in.

Let's talk about a Post model, where you are always going to have a post title. Now the best practice for showing titles is to capitalise them.

Which means a post title called `getting started with adonis` should be displayed as `Getting Started With Adonis`. You would say it is easy to achieve it by using one of the naive techniques.

1. Use CSS property `text-transform` to capitalise it. What if you also have a JSON API?
2. Whenever you find an article, manually capitalise it, by modifying the property.
    - Will do it for 20 posts inside a loop?
    - What if you are fetching posts as a relation for a given user. It means loop through all the users and then their posts and manually mutate the article title. :SHOCKED:

Above tricks are not maintainable. The best way is, to modify the title from its origin. In Lucid, we call them `getters`.

```javascript
class Post extends Lucid {

    getTitle (title) {
        return title
            .toLowerCase()
            .replace(/^(.)|\s(.)/g, function($1) {
                return $1.toUpperCase();
            })
    }

}
```

Now no matter what, the returned title will be the returned value of the above function. Isn't it sweet.

## Getters

Getters are defined as methods on your model. It should start with a keyword called `get` followed by the camel case version of the field name. For example

| Getter | Field Name |
|--------|--------|
getUserName | user_name
getTitle | title
getFirstname | firstname

Following are the traits of a getter method.

1. Getter method will receive the current value of a given field.
2. It is not an asynchronous method, which means you cannot perform time taking operations.
3. Getters are evaluated when you call `.toJSON` method on a model instance or a collection.

## Setters

Setters are opposite of getters, and they mutate the value when you set them on your model instance. For example

```javascript
class User extends Lucid {

 setAccess (access) {
     return access === 'admin' ? 1 : 0
 }

}

const user = new User()
user.access = 'admin'

console.log(user.access) // will return 1
yield user.save()
```

They follow the same convention as getter methods but with `set` as a prefix.


| Setter | Field Name |
|--------|--------|
setUserName | user_name
setTitle | title
setFirstname | firstname


Following are the traits of a setter method.

1. Setter method will receive the current value of the given field.
2. It is not an asynchronous method, which means you cannot perform time taking operations.
3. It is executed every time you set/update a value in the model instance.
4. Setters will not be executed for bulk updates.


## Computed properties.

Computed properties are like getters, but they are virtual values that do not exist in your database tables.

You may want computed properties in many cases. For example calculating the full name of a given user using their first and last name.

```javascript
class User extends Lucid {

  static get computed () {
    return ['fullname']
  }

  getFullname () {
    return `${this.firstname} ${this.lastname}`
  }

}
```

Now whenever you call `.toJSON` on a collection or a user instance, it will return you the fullname.

Computed properties have same traits as getters. However, you have to define the computed properties to create, inside an array.

## FAQ

1. **How to encrypt password, since getters/setters are not asynchronous?**
Password encryption should be the part of `model hooks`. You can have a `beforeCreate` hook to encrypt the password for you.

2. **Why i do not see mutated values when i do `console.log` on my model instance?**
Values on your model instance are the values loaded from the database or value defined by you. You cannot change them using getters. Instead when you call `.toJSON` you get a plain object with your mutated data.