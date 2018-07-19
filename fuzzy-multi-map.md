---
layout: page
title: Fuzzy MultiMap
---

The `FuzzyMultiMap` is basically a [`FuzzyMap`]({{ site.baseurl }}/fuzzy-map) that accepts more than one value per key and stores the colliding items in buckets.

```js
var FuzzyMultiMap = require('mnemonist/fuzzy-multi-map');
```

## Use case

Let's say we need to store items in an index using a fuzzy key so that we may match them against not so precise queries.

Furthermore, since the key produced might be the same for several items and because we want to keep multiple values for each key, we could use a `FuzzyMultiMap` to help us in this task.

```js
// Using lodash to create a naive fuzzy key using the given title
function fuzzyTitle(title) {
  return _.words(title.toLowerCase()).sort().join(' ');
}

// Creating our index
var map = new FuzzyMultiMap(fuzzyTitle);

// Adding some universities
var universities = [
  {name: 'University of Carolina'},
  {name: 'Carolina, university of.'},
  {name: 'Harvard university'}
];

universities.forEach(function(university) {
  map.set(university.name, university);
});

// Now we can query the index
universities.get('university of carolina');
>>> [
  {name: 'University of Carolina'},
  {name: 'Carolina, university of.'}
]
```

## Constructor

The `FuzzyMultiMap` either takes a single argument being a hash function that will process both inserted items or keys & the queries; or a tuple containing two hash functions, one for the inserted items or keys and the second one for the queries.

As with the [MultiMap]({{ site.baseurl }}/multi-map), the `FuzzyMultiMap` can also take a container as second argument.

*Example with one hash function*

```js
// Let's create an index using a single hash function:
var map = new FuzzyMultiMap(function(value) {
  return value.toUpperCase();
});

// Then you'll probably use #.set to insert items
map.set(movie.title, movie);
map.get(queryTitle);
```

*Example with two hash functions*

```js
// Let's create an index using two different hash functions:
var map = new FuzzyMultiMap([
  
  // Hash function for inserted items:
  function(movie) {
    return movie.title.toLowerCase();
  },

  // Hash function for queries
  function(query) {
    return query.toLowerCase();
  }
]);

// Then you'll probably use #.add to insert items
map.add(movie);
map.get(queryTitle);
```

*Example with Set containers*

```js
var map = new FuzzyMultiMap(function(value) {
  return value.toUpperCase(); 
}, Set);
```

**Warning!**: the index will not consider any falsy key processed by its hash functions.

```js
var map = new FuzzyMultiMap(function(item) {
  return item.title && item.title.toLowerCase();
});

var movie = {year: 1999};

// This will not be indexed on `undefined`
map.set(movie.title, movie);
```

### Static #.from

Alternatively, one can build a `FuzzyMultiMap` from an arbitrary JavaScript iterable likewise:

```js
var map = FuzzyMultiMap.from(list, hashFunction [, useSet=false]);
var map = FuzzyMultiMap.from(list, hashFunction [, Container, useSet=false]);
```

## Members

* [#.dimension](#dimension)
* [#.size](#size)

## Methods

*Mutation*

* [#.add](#add)
* [#.set](#set)
* [#.clear](#clear)

*Read*

* [#.get](#get)
* [#.has](#has)

*Iteration*

* [#.forEach](#foreach)
* [#.values](#values)
* [Iterable](#iterable)

### #.dimension

Number of item containers stored in the map.

```js
var map = new FuzzyMultiMap();
map.set('hello', 3);
map.set('hello', 4);

map.size
>>> 2
map.dimension
>>> 1
```

### #.size

Number of items stored in the map.

```js
var map = new FuzzyMultiMap();
map.add({title: 'Hello World!'});

map.size
>>> 1
```

### #.add

Computes the item's key by hashing the given item using the relevant function then adds the item to the map using the key.

`O(1)`

```js
var map = new FuzzyMultiMap();

map.add({title: 'Great movie', year: 1999});

// In fact, same as doing
var movie = {title: 'Great movie', year: 1999};
map.set(movie, movie);
```

### #.set

Adds an item to the map using the provided key that will be processed by the relevant hash function.

`O(1)`

```js
var map = new FuzzyMultiMap();
var movie = {title: 'Great movie', year: 1999};

map.set(movie.title, movie);
```

### #.clear

Completely clears the map of its items.

```js
var map = new FuzzyMultiMap();
map.add(item);
map.clear();

map.size
>>> 0
```

### #.get

Hash the given key using the relevant function then returns the set of items stored by this key.

`O(1)`

```js
var map = new FuzzyMultiMap(function(string) {
  return string.toLowerCase();
});
map.set('John', {name: 'John', surname: 'Williams'});
map.set('John', {name: 'John', surname: 'Ableton'});

map.get('john');
>>> [
  {name: 'John', surname: 'Williams'},
  {name: 'John', surname: 'Ableton'}
]
```

### #.has

Test whether the provided key, processed by the relevant hash function, would return a container.

`O(1)`

```js
var map = new FuzzyMultiMap(function(string) {
  return string.toLowerCase();
});
map.set('John', {name: 'John', surname: 'Williams'});
map.set('John', {name: 'John', surname: 'Ableton'});

map.get('john');
>>> true
```

### #.forEach

Iterates over the values stored in the map.

```js
var map = new FuzzyMultiMap(function(string) {
  return string.toLowerCase();
});

map.set('John', {name: 'John', surname: 'Williams'});
map.set('John', {name: 'John', surname: 'Ableton'});

map.forEach(function(value) {
  console.log(value);
});
>>> {name: 'John', surname: 'Williams'}
>>> {name: 'John', surname: 'Ableton'}
```

### #.values

Creates an iterator over the map's values.

```js
var map = new FuzzyMultiMap(function(string) {
  return string.toLowerCase();
});

map.set('John', {name: 'John', surname: 'Williams'});
map.set('John', {name: 'John', surname: 'Ableton'});

var iterator = map.values();
iterator.next().value();
>>> {name: 'John', surname: 'Williams'}
```

### Iterable

Alternatively, you can iterate over an map' values using ES2015 `for...of` protocol:

```js
var map = new FuzzyMultiMap(function(string) {
  return string.toLowerCase();
});

map.set('John', {name: 'John', surname: 'Williams'});
map.set('John', {name: 'John', surname: 'Ableton'});

for (var value of map) {
  console.log(value);
}
```