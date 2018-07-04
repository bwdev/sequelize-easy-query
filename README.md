# sequelize-easy-query
An easy and robust way of making filtering, searching and ordering using querystring in sequelize.

[![Build Status](https://travis-ci.org/77Vincent/sequelize-easy-query.svg?branch=master)](https://travis-ci.org/77Vincent/sequelize-easy-query)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Installation
```bash
npm install sequelize-easy-query --save
```

## Usage
Let's say we have a "User" table, we want to implement filtering, ordering and searching using querystring, with the native sequelize "where" and "order" clause.
```js
// user-model.js
// For demonstration purpose, some codes are omitted

const Sequelize = require('sequelize')

module.exports.User = new Sequelize(configs).define('user', {
  gender: Sequelize.BOOLEAN,
  active: Sequelize.BOOLEAN,
  age: Sequelize.TINYINT,
  motto: Sequelize.STRING,
  bio: Sequelize.STRING,
  updated_at: Sequelize.Date,
})
```
```js
// user-router.js

const seq = require('sequelize-easy-query')

const users = await User.findAll({
  where: seq('raw query string', {
    filterBy: ['gender', 'active'],
    searchBy: ['bio', 'motto'],
    orderBy: ['age', 'updated_at'],
  }),
})
```

## Table of API
##### Basic query
* [filterBy](#filterBy)
* [searchBy](#searchBy)
* [orderBy](#orderBy)
##### Query with alias
* [filterByAlias](#filterByAlias)
* [orderByAlias](#orderByAlias)
##### Pre-query
* [filter](#filter)
* [search](#search)
* [order](#order)

### <a name="filterBy"></a>filterBy
To filter the "User" table by "gender" and "active" column, simply do:
```js
const users = await User.findAll({
  where: seq('raw query string', {
    filterBy: ['gender', 'active'],
  }),
})
```
Now you can filter with querystring individually or in combination:
```bash
example.com/api/users?gender=0&active=1
```
Multiple selection for the same column:
```bash
example.com/api/users?gender=0&gender=1
```

### <a name="searchBy"></a>searchBy
To search the "User" table by content in "bio" **OR** "motto" column, simply do:
```js
const users = await User.findAll({
  where: seq('raw query string', {
    searchBy: ['bio', 'motto'],
  }),
})
```
Now you can trigger a search by using the key "search", which will give you those users that have "some_values" in their "bio" **OR** "motto" field:
```bash
example.com/api/users?search=some_values
```
Unlike filterBy, multiple search is **NOT SUPPORTED** yet:
```bash
example.com/api/users?search=some_values&search=some_other_values
```

### <a name="orderBy"></a>orderBy
To order the "User" table by "age" or "updated_at" column, simply do:
```js
const users = await User.findAll({
  order: seq('raw query string', {
    orderBy: ['age', 'updated_at'],
  }),
})
```
Now you can order the table by "age" **OR** "updated_at" respectively, only two options are usable: DESC or ASC:
```bash
example.com/api/users?age=DESC
```
```bash
example.com/api/users?updated_at=ASC
```
Multiple ordering is meaningless, only the first query will work:
```bash
example.com/api/users?age=DESC&updated_at=ASC
```

### <a name="filterByAlias"></a>filterByAlias
Sometimes if you want the key used for query to not be the same as its corresponding column name, you can do:
```js
const users = await User.findAll({
  order: seq('raw query string', {
    filterBy: {
      gender: 'isMale',
      active: 'isAvailale',
    },
  }),
})
```
Now you can filter users by using the new keys and the original ones can no longer be used:
```bash
example.com/api/users?isMale=0&isAvailable=1
```
This feature is especially useful when you have included other associated models, you want to filter the main model by some columns from those associated models but to not affect the main model:
```js
const users = await User.findAll({
  include: [{
    model: Puppy,
    where: seq('raw query string', {
      filterByAlias: {
        gender: 'puppy_gender'
      }
    })
  }],
  where: seq('raw query string', {
    filterBy: ['gender']
  }),
})
```
Now "puppy_gender" is used to filter users based on their puppies' gender, but not they themselves' gender:
```bash
example.com/api/users?puppy_gender=1
```
While "gender" is still used to filter users by users' gender:
```bash
example.com/api/users?gender=1
```

Alias can also be given the same value as the original column name, it's totally fine:
```js
const users = await User.findAll({
  where: seq('raw query string', {
    filterBy: {
      gender: 'gender',
      active: 'active',
    },
  }),
})
```
Then everything will be just like using filterBy:
```bash
example.com/api/users?gender=0&active=1
```

### <a name="orderByAlias"></a>orderByAlias
Please refer to [filterByAlias](#filterByAlias) which is for the same purpose and with the same behaviour.

### <a name="filter"></a>filter
If you want to pre-filter the table, simply do:
```js
const users = await User.findAll({
  where: seq('raw query string', {
    filter: {
      gender: 1,
      active: 0,
    }
  }),
})
```
Now without actually passing any querystring, you will get result that's already been filtered:
```bash
example.com/api/users
```

### <a name="search"></a>search
If you want to pre-search the table, simply do as follow, to be noticed that "searchBy" is needed to be declared as it tells database which column to search:
```js
const users = await User.findAll({
  where: seq('raw query string', {
    search: 'content to search',
    searchBy: ['bio', 'motto'],
  }),
})
```
Now without actually passing any querystring, you will get result that's already been searched:
```bash
example.com/api/users
```

### <a name="order"></a>order
If you want to pre-order the table, simply do as follow, to be notice that only one column 
```js
const users = await User.findAll({
  order: seq('raw query string', {
    order: {
      age: 'DESC'
    }
  }),
})
```
Now without actually passing any querystring, you will get result that's already been searched:
```bash
example.com/api/users
```

