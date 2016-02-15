# Minification safe Dependency Injection

## Overview

Dependency Injection is great but we can bite ourselves in the foot if we aren't careful. Let's have a look at how dependency injection can fail, and what we do to make things sweet again.

## Objectives

- Describe Angular DI annotations
- Use Array annotations syntax
- Use $inject annotations syntax

## Minification

Minification is great - it reduces our code's file size and allows us to code wonderful things without impacting load times. It strips out all unneeded whitespace and punctuation in order to make our files as small as they can possibly be.

For instance, a minifier might take this:

```js
function add(numberOne, numberTwo) {
	return numberOne + numberTwo;
}
```

And change it into:

```js
function add(a,b){return a+b;}
```

Much smaller! However, it can cause a bit of an issue when mixed with dependency injection.

Most minify-ers change our variable names to single letters - it isn't important what our variables are called, only that they all still refer to the same object/method/function/etc. For example:

```js
function ContactController($scope, $timeout) {
}

angular
  .module('app')
  .controller('ContactController', ContactController);
```

Would become:

```js
function a(b, c) {
}

angular
  .module('app')
  .controller('ContactController', a);
```

Oh no - we don't have services named `b` and `c`. Therefore, when Angular looks at our controller and notices that we require `b` and `c`, it'll throw an error as they don't exist.

How do we overcome this? Well, it's important to know how dependency injection works under the hood first.

## Under the hood

When Angular looks at our function, it takes our parameters and creates an array of what we require. For instance, our controller above would generate the array `['$scope', '$timeout']`. Angular then attaches this array onto a property named `$inject` on our function.
 
```js
function ContactController($scope, $timeout) {
}

angular
  .module('app')
  .controller('ContactController', ContactController);
  
// ContactController.$inject = ['$scope', '$timeout'];
```

Can you see how we can resolve the issues when it comes to minification? We can take over from Angular and specify this `$inject` property ourselves. When the `$inject` property exists, Angular trusts us to know what we want and doesn't look at the controller's parameters - meaning we can name our parameters whatever we want.
 
```js
function ContactController(whatever, weWant) {
  // whatever === $scope
  // weWant === $timeout
}

ContactController.$inject = ['$scope', '$timeout'];

angular
  .module('app')
  .controller('ContactController', ContactController);
``` 

In our example above, the variable `whatever` is still equal to the value of `$scope` - just named differently! The same applies to the variable `weWant`. This means that our minification process will have no side effects to our code.