# Javascript Lessons Learned

## An interesting pattern for argument "splatting".

We can make clever use of javascripts `Function.prototype.apply` to emulate argument "splatting" in javascript.

```js
var fn = function(arg1, arg2, arg3, arg4) {
  console.log(arg1, arg2, arg3, arg4);
}
var args = [1, 2, 3];
fn.apply(null, args); // => 1, 2, 3, undefined
```

Interestingly if we are using ES6 this behavior is now built in. It is called the _spread operator_

```js
var fn = function(arg1, arg2, arg3, arg4) {
  console.log(arg1, arg2, arg3, arg4);
}

var args = [1, 2, 3];
fn(...args);
```

Using babeljs, this ES6 idiom actually transpiles down to `fn.apply(undefined, args)` so we can be confident that this is the way to do splatting currently in es5.
