# mori

<img src="http://cloud.github.com/downloads/swannodette/mori/mori.png" alt="Mori" title="Mori"/>

A simple bridge to ClojureScript's persistent data structures and [supporting APIs](http://swannodette.github.io/mori/) for vanilla JavaScript. Pull requests welcome.

## Getting it

You can install the latest release via npm:

```shell
npm install mori
```

The installed package contains a single optimized JavaScript file `mori.js`.

Load `mori` in your Node.js programs as you would any other module:

```javascript
var mori = require("mori");
```

In a browser, you can load mori with a script tag, as you would any other JavaScript library:

```html
<script src="mori.js" type="text/javascript"></script>
```

You can also load it as an AMD module, e.g. with [RequireJS](http://requirejs.org/).

## Build

### Prerequisites

You will first need to install the [Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html) SDK, if it's not already installed on your system.

On Windows, you will need to manually install [Leiningen](http://github.com/technomancy/leiningen). On UNIX-like systems, Leiningen will be installed within the project automatically if the `lein` executable is not found on your path or if your `lein` version predates `2.0.0`.

### Clone the repo

```shell
git clone https://github.com/swannodette/mori.git
cd mori
```

### On a UNIX-like system build with

```shell
./scripts/build.sh
```

### Alternatively using npm

```shell
npm run-script build
```

### On Windows

```shell
./scripts/build.ps1
```

The build process will generate an optimized JavaScript file `mori.js`, which is suitable for use with Node.js, or in a Web browser or other JavaScript environments. You can also load it as an AMD module.

## Usage

You can use it from your projects like so:

```javascript
var inc = function(n) {
  return n+1;
};

mori.into_array(mori.map(inc, mori.vector(1,2,3,4,5)));
// => [2,3,4,5,6]
```

Efficient non-destructive updates!

```javascript
var v1 = mori.vector(1,2,3);
var v2 = mori.conj(v1, 4);
v1.toString(); // => '[1 2 3]'
v2.toString(); // => '[1 2 3 4]'
```

```javascript
var sum = function(a, b) {
  return a + b;
};
mori.reduce(sum, mori.vector(1, 2, 3, 4)); // => 10
```

Lazy sequences!

```javascript
var _ = mori;
_.into_array(_.interpose("foo", _.vector(1, 2, 3, 4)));
// => [1, "foo", 2, "foo", 3, "foo", 4]
```

Or if it's more your speed, use it from CoffeeScript!

```coffeescript
inc = (x) -> x+1  
r = mori.map inc, mori.vector(1,2,3,4,5)
mori.into_array r
```

### Documentation

You can find extensive [documentation and examples](http://swannodette.github.io/mori/) here.

## More Examples

### Efficient Freeze/Thaw

For vectors and maps we provide an efficient thaw and freeze
operations:

```javascript
var m = mori;

// ~330ms with v8 3.22.11 MBA 1.7ghz
for(var j = 0; j < 10; j++) {
  var s = new Date();
  var arr = [];
  for(var i = 0; i < 10000000; i++) {
    arr.push(i);
  }
  print("Array push " + arr.length + " items " + ((new Date())-s));
  gc();
}

// ~360ms
for(var j = 0; j < 10; j++) {
  s = new Date();
  var mv = m.mutable.thaw(m.vector());
  for(var i = 0; i < 10000000; i++) {
    mv = m.mutable.conj1(mv, i);
  }
  var v = m.mutable.freeze(mv);
  print("Mutable vector conj " + m.count(v) + " items " + ((new Date())-s));
  gc();
}
```

### ES6 Map/Set inspired interfaces

All Mori maps and sets support all the non-mutating methods of the
proposed ES6
[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
and
[Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
interfaces. The main difference with the spec is that key lookup is
based on value not reference. `keys`, `values`, and `entries` methods
return the proposed mutable iterators:

```javascript
var m = mori;
var h = m.hash_map("foo", 1, "bar", 2);

h.has("foo"); // => true
h.get("foo"); // => 1

var iter = h.keys();
iter.next(); // => {done: false, value: "foo"}
```

This feature is subject to changes in the ES6 proposal.

### Reducers

Mori includes the new Clojure reducers framework. Zero allocation collection operations FTW:

```javascript
var m = mori;
var a = [];

for(var i = 0; i < 1000000; i++) {
  a.push(i);
}

// make it immutable
var v = m.into(m.vector(), a);

var mul3 = function(n) {
  return n*3;
}

function time(f) {
  var s = new Date();
  f();
  console.log(((new Date())-s)+"ms");
}

// 87ms 3.5ghz iMac recent V8 build
time(function() {
  m.reduce(m.sum, 0, m.rmap(m.inc, m.rmap(m.inc, m.rmap(m.inc, v))));
});

// 365ms
time(function() {
  a.map(m.inc).map(m.inc).map(m.inc).reduce(function(a,b){return a+b;}, 0);
})
```

### Pipelines

```javascript
mori.pipeline(mori.vector(1,2,3),
              function(v) { return mori.conj(v,4) },
              function(v) { return mori.drop(2, v) });

// => [3 4]
```

### Currying

```javascript
mori.pipeline(mori.vector(1,2,3),
              mori.curry(mori.conj, 4),
              mori.curry(mori.conj, 5));

// => [1 2 3 4 5]
```

### Partial Application

```javascript
mori.pipeline(mori.vector(1,2,3),
              mori.curry(mori.conj, 4),
              mori.partial(mori.drop, 2));

// => (3 4)
```

### Function Composition

```javascript
var second = mori.comp(mori.first, mori.rest);

second(mori.vector(1,2,3));
// => 2
```

### Juxtaposition

```javascript
var pos_and_neg = mori.juxt(mori.identity, function (v) { return -v; });
pos_and_neg(1);
// => [1 -1]

mori.knit(mori.inc, mori.dec)(pos_and_neg(1));
// => [2 -2]
```


Copyright (C) 2014 David Nolen and contributors

Distributed under the [Eclipse Public License](https://raw.github.com/swannodette/mori/master/epl-v10.html), the same as Clojure.
