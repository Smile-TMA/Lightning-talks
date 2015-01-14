# Constructor and prototype

## Foreword

This files contains examples illustrating [this presentation](https://prezi.com/qtvjlm_557ab/javascript-constructor-and-prototype/).

You can use the chrome console for the above examples (avoid firebug).

Presentation time needed : 30-40min

## `this`

### Value of `this` within a classic function

Example :
```javascript
function f() {
  return this;
}
```

Result :
```javascript
f();
```

### Value of `this` within a classic function in strict mode

Example :
```javascript
function f_strict() {
  'use strict';
  return this;
}
```

Result :
```javascript
f_strict();
```

### Other value of `this` (event callback)

Example :
```javascript
var a = document.createElement('a');
a.href = 'http://www.google.fr';
a.onclick = function () {
  console.log(this);
  return false;
};
a.appendChild(document.createTextNode('Click me !'));
document.getElementsByTagName('body')[0].appendChild(a);
```

Result : click on the link to see it.

## Object literals methods

### Value of `this` within a method

```javascript
var point = {
  x : 5,
  y : 2,
  toString : function () {
    return this.x + ',' + this.y;
  }
};
```

Result :
```javascript
point.toString();
```

### Extracted method loses binding

Example :
```javascript
var toString = point.toString;
```

Result :
```javascript
toString();
```

### Each function has its own `this`

Example :
```javascript
point.dist = function () {
  var square = function(prop) {
    return this[prop] * this[prop];
  };

  return Math.sqrt(square('x') + square('y'));
};
```

Result :confused: :
```javascript
point.dist();
```

### `dist` workaround 1

Example :
```javascript
point.dist = function () {
  var square = function(value) {
    return value * value;
  };

  return Math.sqrt(square(this.x) + square(this.y));
};
```

Result :
```javascript
point.dist();
```

### `dist` workaround 2

Example :
```javascript
point.dist = function () {
  return Math.sqrt(this.square('x') + this.square('y'));
};
point.square = function(prop) {
  return this[prop] * this[prop];
};
```

Result :
```javascript
point.dist();
```

## `bind`, `call` and `apply`

### `bind` with extracted method

Example :
```javascript
var toString = point.toString.bind(point);
```

Result :
```javascript
toString();
```

### Using `bind` for `dist` workaround 1

Example :
```javascript
point.dist = function () {
  var square = function(prop) {
    return this[prop] * this[prop];
  }.bind(this);

  return Math.sqrt(square('x') + square('y'));
};
```

Result :
```javascript
point.dist();
```

### Using `bind` for `dist` workaround 2

Example :
```javascript
point.dist = function () {
  return Math.sqrt(
    Object.keys(this)
      .filter(function(prop){
        return typeof this[prop] == 'number';
      }.bind(this))
      .map(function(prop) {
        return this[prop] * this[prop];
      }.bind(this))
      .reduce(function(a, b){
        return a + b;
      })
  );
};
```

Result :
```javascript
point.dist();
```

pros :
* Can work with an infinite amount of properties (x , y, z...).

cons :
* Too complex to understand. It should be rewritten to be clearer. This was done on purpose for this example.

Explanations :
* `Object.keys(this)` returns `["x", "y", "toString", "dist"]`
* `filter(...)` returns `["x", "y"]`
* `map(...)` returns `[25, 4]`
* `reduce(...)` returns `29`
* `Math.sqrt()` return `5.385164807134504` :metal:

### Using `call` for `dist` workaround

Example :
```javascript
point.dist = function () {
  var square = function(prop) {
    return this[prop] * this[prop];
  };

  return Math.sqrt(square.call(this, 'x') + square.call(this, 'y'));
};
```

Result :
```javascript
point.dist();
```

### Using `apply` for `dist` workaround

Example :
```javascript
point.dist = function () {
  var square = function(prop) {
    return this[prop] * this[prop];
  };

  return Math.sqrt(square.apply(this, ['x']) + square.call(this, ['y']));
};
```

Result :
```javascript
point.dist();
```

### Exercice : create a function that adds each of its argument each other

Try :
```javascript
var add = function() {
  return arguments.reduce(function(a, b){
    return a + b;
  });
};
```

Result :worried: :
```javascript
add(1, 2, 3, 4);
```

Explanations : The `arguments` object is an [Array-like object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Functions/arguments), thus array's properties are not all available.

You can run this function to see what array's properties are available for `arguments` :
```javascript
(function() {
  var properties = Object.getOwnPropertyNames(Array.prototype);
  for (var i in properties) {
    console.log(properties[i] + ' : ' + (properties[i] in arguments));
  }
})()
```

Explanations :
* `Object.getOwnPropertyNames` lists all properties (that are not inherited) belonging to `Array.prototype`.
  It is used to list all properties, methods applicable to arrays.
* `(properties[i] in arguments)` check if the property exist for `arguments`.

### `add` workaround

Example :
```javascript
var add = function() {
  return Array.prototype.reduce.call(arguments, function(a, b){
    return a + b;
  });
};
```
Result :sunglasses: :
```javascript
add(1, 2, 3, 4);
```

### `bind`, `call` and `apply` at the same time for `dist` workaround :smiling_imp: :smiling_imp: :smiling_imp:

Example :
```javascript
point.dist = function () {
  var square = function(prop) {
    return this[prop] * this[prop];
  };

  var add = function() {
    return Array.prototype.reduce.call(arguments, function(a, b){
      return a + b;
    });
  };

  return Math.sqrt(add.apply(this, ['x', 'y'].map(square.bind(this))));
};
```
Result :triumph: :
```javascript
point.dist();
```

Ok that's just over-engineering..

![Over-engineering is bad](http://i.imgur.com/JPsizDt.jpg)

But that's fun :stuck_out_tongue:.

## Constructor

### Simple constructor

Example :
```javascript
var Point = function(){
  this.x = 5;
  this.y = 2;
  this.toString = function () {
    return this.x + ',' + this.y;
  }
};
```

Result :
```javascript
var point = new Point();
point + '';
point instanceof Point;
```

In fact, it's just the same as what we had earlier if you write it like this (like an object factory) :
```javascript
var Point = function(){
  return {
    x : 5,
    y : 2,
    toString : function () {
      return this.x + ',' + this.y;
    }
  };
};
```

But it's no more considered as an instance of `Point` :
```javascript
var point = Point();
point + '';
point instanceof Point;
```

## Prototype

### Used to save memory

Example :
```javascript
var Point = function(){
  this.x = 5;
  this.y = 2;
};
Point.prototype = {
  toString : function () {
    return this.x + ',' + this.y;
  }
};
```

Result :
```javascript
var point = new Point();
point + '';
```

### The change of a constructor prototype method will impact all instances

Example :
```javascript
var Point = function(){};
Point.prototype = {
  x : 5,
  y : 2,
  toString : function () {
    return this.x + ',' + this.y;
  }
};
```

```javascript
var point1 = new Point();
var point2 = new Point();
Point.prototype.y = 3;
point1.y;
point1.__proto__.y = 4;
point2.y;
```

Conclusion : Use prototype for methods and not for properties (use constructor instead).

### The constructor prototype property

Example :
```javascript
var Point = function(){
  this.x = 5;
  this.y = 2;
};
```

Result :
```javascript
Point.prototype.constructor;
Point === Point.prototype.constructor;
```

If you define the prototype as an objec literal you will lose the constructor property.

So it's better to define each methods separately :
```javascript
Point.prototype.toString = function () {
  return this.x + ',' + this.y;
};
```

## inheritance

### Using `Object.create`

Example :
```javascript
var Point = function(){
  this.x = 5;
  this.y = 2;
};
Point.prototype.toString = function () {
  return this.x + ',' + this.y;
};
```

Result :
```javascript
point1 = new Point();
point2 = Object.create(Point.prototype);
point1.__proto__ === point2.__proto__;
```

`Object.create` is used to create an object whose prototype will be equal to its argument.

But comparing to `point1`, `point2` has not been initialized through the constructor (and thus is not an instance of `Point`).

### Simple inheritance example

Defining the `Point2D` constuctor and prototype :
```javascript
var Point2D = function(){
  this.x = 5;
  this.y = 2;
};
Point2D.prototype.toString = function () {
  return this.x + ',' + this.y;
};
Point2D.prototype.dist = function () {
  return Math.sqrt(
    Object.keys(this)
      .filter(function(prop){
        return typeof this[prop] == 'number';
      }.bind(this))
      .map(function(prop) {
        return this[prop] * this[prop];
      }.bind(this))
      .reduce(function(a, b){
        return a + b;
      })
  );
};
```

Defining the `Point3D` constuctor  :
```javascript
var Point3D = function(){
  this.x = 1;
  this.y = 2;
  this.z = 3;
};
```

Extending `Point3D` prototype from  `Point2D` prototype :
```javascript
Point3D.prototype = Object.create(Point2D.prototype);
```

Rewriting the `toString` method using the "parent" method (no native shortcut) :
```javascript
Point3D.prototype.toString = function () {
  return Point2D.prototype.toString.apply(this) + ',' + this.z;
};
```

Result :
```javascript
point3d = new Point3D();
point3d.dist();
point3d + '';
point3d instanceof Point3D;
point3d instanceof Point2D;
point3d instanceof Object;
point3d;
```