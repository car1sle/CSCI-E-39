# Lecture 1: ES6 Primer

JS has been around for a long time, all the way back to 1995. As such, there are many versions of it so some may be familiar with older syntax and data structures. In this lecture we'll review what's new in the latest versions so when you find examples or browse a code base you're not surprised with some strange unknown syntax.

## Variables

JS is not a typed language, that is, when you define a variable, you're not required to define it's data type. We can create a variable very easily and have it hold any type of value:

```JS
var a = 1

// Now change the same variable to a different type
a = 'I am now a string!'

// And change it again
a = {
  now: 'I am',
  an: 'object'
}
```

All of the above can be executed within the same script and wouldn't throw any errors.

ES6 introduced two new ways to define variables that are still not typed, but provide at least some restrictions in what you can do with them.

`let`: defines a variable that can be reassigned
`const`: creates a read-only variable which can only be assigned once to an initial value.

### Understanding Scope

Scope is a region of a program and it's very important to understand where the variables (or name bindings) that we define are valid and available. Scope bugs are hard to debug and fix!

`var` declarations are scoped to the *function* where they are defined

`let` and `const` have *block* scope, which is more inline with most programming languages.

Lets look at some examples:

```JS
var outerVar = 1;
let outerLet = 1;

function modify1() {
  // These changes will modify the outer variables
  outerVar = 2;
  outerLet = 2;
}

modify1()

console.log(outerVar, outerLet);
// Output: 2, 2,
// Because we're modifying the variables defined in the outer scope

function modify2() {
  var outerVar = 3;
  let outerLet = 3;
}

modify2()
console.log(outerVar, outerLet);
// Output: 2, 2
// Because we have defined new variables within the modify2 function. The modifications are only local to the function and not outside.

if (1 === 1) {
  var outerVar = 4;
  let outerLet = 4;
}

console.log(outerVar, outerLet);
// Output: 4, 2
// For outerVar, we have re-declared the variable in the function (script) scope. However, the outerLet defined within the {} of the if statement is a new variable that only lives within that scope, leaving the outer one untouched
```

## Functions

ES6 introduced new syntax to declare functions. Some key differences:

### `arguments`

In old JS, every function receives a special argument called `arguments`. This was simply an object (or a list of `key/value` pairs) that was handy in some scenarios to know everything that was available in the function. This is not available anymore.

```JS
function oldFunction(a, b) {
  console.log('here!', JSON.stringify(arguments), a, b);
  console.log(this) // -> prints execution context (global)
}

oldFunction(1, 2);

// Output: 
// here! {"0":1,"1":2} 1 2

arguments = null
const newFunction = (a, b) => {
  console.log('here!', arguments, a, b)
  console.log(this) // -> prints local context (undefined)
}
```

## `this`

The usage of `this` is widespread in JS and in general is not something we recommend relying on. It leads to a lot of errors because `this` can have lots of different values depending on context and even the syntax used. Example:

```JS
const obj = {
  name: 'Inner',
  arrow: () => {
    console.log('Arrow:\t\t' + this.name);
    console.log(this);
    // Here "this" holds the inner function context
    // which is an empty object ({}) and hence this.name
    // is undefined
  },
  traditional() {
    console.log('Traditional:\t' + this.name);
    console.log(this);
    // Here "this" is a reference to the object, so 
    // we do have access to the name
  },
};

obj.arrow();
obj.traditional();
```

## Destructuring

This is simply a new syntax to extract data from objects. It is extremely helpful, but sometimes counterintuitive and has some caveats. Note that destructuring != destructing!

```JS

const destructure = {
  a: 1,
  b: {
    c: 2,
    d: {
      e: 3,
    },
  },
};

// This pulls the value of 'a' from the object
const { a } = destructure;

// We can go further
const {
  b: { c },
} = destructure;

console.log(c);
console.log(b); // Note that b is not defined in this case

// If we wanted to pull both `b` and `c`, we would do
const {
  b,
  b: { c },
} = destructure;
```

We can also destruct arrays:

```JS
const arr = [1, 2, 3];
const [first] = arr;
const [first, second] = arr;
const [, , third] = arr;

const test = 'abc';
const [a] = test;
```

If we have objects that have the same keys, we cannot destructure them at the same time because we would have variables with the same name. Instead, we have to rename the variable:

```JS
const o1 = { a: 1 };
const o2 = { a: 1 };
const { a: o1_a } = o1;
const { a: o2_a } = o1;
```

Sometimes destructuring can be overused:

```JS
// Go crazy
const crazy = [
  1,
  2,
  {
    a: 1,
    b: ['x', 'y', 'z', 'a'],
  },
];

// Extract 'y' and assign it to a variable called myY
const [
  ,
  ,
  {
    b: [, myY],
  },
] = crazy;

// The equivalent in traditional JS would be simply:
var myY = crazy[2].b[1];
```

One excellent use case for destructuring is function arguments. Imagine a function that receives 3 arguments:

```JS
const func = (a, b, c) => {
  console.log(a, b, c);
}
```

When I'm using the function, it's not immediately clear what that last argument is for. Seems to be a flag to do something but unclear what it is. Further, what if I want to call this function without the second argument?

```JS
// What is "false" for?
func(1, 'argument 2', false);

// The only way to "omit" argument 2 is:
func(1, null, false);
func(1, undefined, false);
```

But we can make this much better if we redeclare our function:

```JS
// The function receives a single arguments object
const func = arguments => {
  // Destruct the arguments
  const { a, b, c } = arguments;
  console.log(a, b, c);
}
```

so now we can call it as:

```JS
const args = {
  a: 1,
  b: 'argument 2',
  c: false,
};

func(args);
```

now it's very clear what each value is for. Further, we can simplify this as:

```JS
// Destruct the arguments directly in the function signature
const func = ({ a, b, c }) => {
  console.log(a, b, c);
}

// And the call as:
func({
  a: 1,
  b: 'argument 2',
  c: false
});

// Omiting an argument is as simple as:
func({
  a: 1,
  c: false
});
```

### Spread Operator

Sometimes we need to destruct an object to pull out some properties but we still want to keep the rest. We can do this using the spread operator:

```JS
const spread = {
  a: 1,
  b: 2,
  c: 3,
  d: 4,
};

const { a, b, ...rest } = spread;

// In Arrays
const spreadArr = [1, 2, 3, 4];
const [f1, f2, ...rest] = spreadArr;
const newArr = ['a', 'b', ...spreadArr];
```

We can also use the spread operator to "clone" objects:

```JS
// Cloning objects
const sheep = {
  a: 1,
  b: 2,
  c: {
    d: 3,
  },
};

const clone = { ...sheep };

// If we modify the 'a' key for the clone, the original remains untouched:
clone.a = 'modA';
console.log(sheep.a, clone.a);

// However, note that the spread operator is a shallow copy of an object. That is, it will copy the values of the top level keys. In this case, the 'c' key is a refence to an object, so the clone has a reference to this same object:

clone.c.d = 'modD';
// These two values will be the same:
console.log(sheep.c.d, clone.c.d);
```

## Promises

Promises were created to avoid callback hell. That is, passing in a function to be "called back" when an asynchronous operation finished.

```JS
const p = ({ arg1, arg2, callback }) => {
  setTimeout(() => {
    callback('done');
  }, 5000);
};
```

With promises, this becomes much easier to write, albeit sometimes more confusing:

```JS
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('done');
  }, 5000);
});

p.then(r => {
  console.log(r);
}).catch(e => {
  console.log(e);
});
```

One common misconception is thinking that the promise body only gets executed when its "then" is requested. However, the Promise body gets executed immediately:

```JS
const p = new Promise((resolve, reject) => {
  // This will log even if we don't have the then below
  console.log('started');

  setTimeout(() => {
    console.log('resolve');
    resolve(true);
  }, 1000);

  console.log('return from promise');
});

// This is only necessary if we want to capture the results of the promise, but even if we don't it will still execute
p.then((r) => {
  console.log('success', r);
})
  .catch((e) => {
    console.log('fail', e);
  })
  .finally(() => {
    console.log('done');
  });
```

One of the main benefits of asynchronous operations is that we can run them in parallel. With promises, we can not only run them in parallel but also wait for all of them to finish:

```JS
const p1 = new Promise((resolve, reject) => {
  try {
    resolve('success result 1');
  } catch (e) {
    reject('error result 1', e);
  }
});

const p2 = new Promise((resolve, reject) => {
  try {
    resolve('success result 2');
  } catch (e) {
    reject('error result 2', e);
  }
});

// This will expect ALL promises to succeed, if one fails, the whole thing fails
// the first promise to fail is the one that breaks all of it
Promise.all([p1, p2]).then((r) => {
  // The result is an array that contains the results of each promise in the same order they were waited for
  console.log(r);
});
```

One great thing about the "then" syntax is that we can chain as many as we want, creating a nice construct:

```JS
// Sequencing thens
p1.then((r) => {
  console.log(r);
  return r * 2;
})
  .then((r) => {
    console.log(r);
    return r * 2;
  })
  .then((r) => {
    console.log(r);
    return r * 2;
  })
  .then((r) => {
    console.log(r);
    return r * 2;
  })
  .then((f) => {
    console.log(f);
  });
```

In JS, functions are a first-class citizens, that is, you can pass them around, store them in variables. It is no surprise then that Promises are too!

```JS
const creator = (result) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const res = result * 2;
      console.log(res);
      resolve(res);
    }, 500);
  });
};

// Now we can use our promise creator to create as many promises as needed
const p1 = creator(1)
  .then((r) => {
    return creator(r);
  })
  .then((r) => {
    return creator(r);
  })
  .then((r) => {
    return creator(r);
  });

// Or even shorter syntax (not recommended):
const p1 = creator(1)
  .then(creator)
  .then(creator)
  .then(creator)
  .then(creator)
  .then(creator)
  .then(creator)
  .then(creator)
  .then(creator);
```

### async / await
ES2017 introduced this new syntax to write promises, without writing promises. It's basically a way to write JS in a way that looks synchronous:

```JS
const p1 = () =>
  new Promise((resolve, reject) => {
    console.log('Created 1');
    setTimeout(() => {
      console.log('Done 1');
      resolve('Done 1!');
    }, 1000);
  });

const p2 = () =>
  new Promise((resolve, reject) => {
    console.log('Created 2');
    setTimeout(() => {
      console.log('Done 2');
      resolve('Done 2!');
    }, 1000);
  });

const f = async () => {
  console.log('Started...');
  const a = await p1();
  const b = await p2();
  // Note that p2 only runs after p1 is done, i.e. synchronously
};
```

## Iterators
Most older versions of languages did not provide a lot of data manipulation functions. However, applications today are increasingly requiring more and more data. Ingesting, transforming, converting data is one of the best skills a programmer can have because most apps require heterogeneous data, coming in from lots of different sources. Iterators are some of the best tools you master. Iterators are methods that can be called for any collection we create. We will use a simple array to demo:

```JS
const arr = ['a', 'b', 'c', 'd'];
```

### `forEach`
A simple loop that receives a function to be called on each item and returns nothing:

```JS
const res = arr.forEach(l => {
  console.log(`letter is ${l}`);
});
```

Note that we cannot stop a `forEach` loop the way we can `break` a `for` loop. That is, the function will be executed for EVERY item in the array, regardless of the result of each call. If we do need to break the loop we need to use a traditional `for` loop.

### `map` 
Map is a simple yet powerful construct, used quite often in React and web-applications in general. It is similar for `forEach` but returns an array with the results of each call:

```JS
// Map returns a value for each item
const res = arr.map((letter) => {
  return `letter is ${letter}`;
});

// res now contains a list of strings
```

### `filter`
If we want to remove some items from a list we can use `filter` to pick the ones we need. This construct receives a function that should only return `true` for the items we want to keep:

```JS
const res = arr.filter((l) => l > 'b');
// res now has all the elements that meet this criteria
```

### `find`
Used when we just want to find the first element that meets a certain condition.

```JS
const res = arr.find((l) => l > 'b');
// res now has the first element that meets this criteria
```

### `some`
Slight variation of `find`. Instead of returning the matching item, it just returns `true` if an element matches.

### Chaining Iterators
It is very common to have to apply multiple transformations to a dataset. For the iterators that return collections, these can be easily changed:

```JS
const res = [1, 2, 3, 4, 5]
  .map((x) => x * 3)
  .filter((x) => x % 2 === 0)
  .map((x) => `${x} is even`);
```

### `reduce`
Save the best for last. `reduce` is the most versatile but also most complex iterator. It takes a quite a few tries to master but it is the most powerful and the best one to have in our toolset.
`reduce` receives a function, which gets 4 arguments:

- `accumulator`: where we are "accumulating" our results
- `currentValue`: the current item in our iteration
- `currentIndex`
- `collection`: the original collection

this function, should return the new accumulator with any modifications done based on the current item. The second argument to `reduce` is the initial value of the accumulator.

```JS
const reduced = arr.reduce((acc, curr, currIndex, arr) => {
  if (curr % 2 === 0) {
    return [...acc, `${curr} is even`];
  } else {
    return acc;
  }
}, []);
```

## Default Values
There are many scenarios where we need to assign a default value to a variable if one is not provided.

### For functions
As mentioned above, destructing arguments in functions is extremely handy. But what happens if a function that destruct arguments is called *without* arguments?

```js
const func = ({ a, b }) => {
  return a + b;
}

// This will throw an exception!
func();
```

To fix this, we can assign a default value as:

```js
const func = ({ a, b } = {}) => {
  return a + b;
}

// Now this will work
func();
```

Although this works, it doesn't produce a necessarily valid result since it returns `NaN`. We can adjust to have default values for `a` and `b` respectively as follows:

```js
const func = ({ a, b } = { a: 0, b: 0 }) => {
  return a + b;
}

func();
```

### For variables
There are cases were we need to avoid `undefined` variables, so we need to assign a default value to them. Although we can do this using the `||` operator - if the left hand side is *truthful* that is the value our variable gets, otherwise it resolves to the right hand side. However, some expressions in JS are unexpectedly *falsy*. For example:

```js
const a = 0 || -1; // 0 is considered falsy, so this results in a being -1;
const b = '' || 'default'; // an empty string is also falsy so this results in b being 'default';
```

To avoid these issues, we can resort to the nullish coalescing operator, which returns it's right-hand side operand when the left-hand side is either `null` or `undefined`:

```js
const a = 0 ?? -1; // a is now 0
const b = '' ?? 'default'; // b is now ''
const c = null ?? 'def'; // c is now 'def'
```

### `NULL` vs `undefined`

All languages have a concept of `NULL`, as a placeholder for something that just has no value. In JS, and most non-typed (or dynamically-typed languages), there is also the concept of `undefined`, as a variable that simply has not been *defined*:

```js
let a; // As undefined
let b = null; // b is defined and has a NULL value
```

### The `===` operator
Comparing values is one of the most common operations in a program. In JS, comparing values using the traditional `==` can sometimes lead to unexpected results:

```js
// Expected:
1 == 1      // true
'a' == 'a'  // true
{ a: 1 } == { a: 1 } // false => these are not the same reference!

// Unexpected
1 == '1'    // true
0 == ''     // true => they're both falsy statements, hence they're considered ==
undefined == null // true
```

However, to avoid these confusing examples, we can use the `===` operator, which compares the types of values as well:

```js
1 === 1     // true
'a' === 'a' // true
{ a: 1 } === { a: 1 } // false
1 === '1'   // false
0 === ''     // false
undefined === null // false
```

### Compilation vs Interpretation vs Hoisting
These concepts are quite confusing so we'll start with defining them.

**Compilation**: Before a compiled language can be executed by a machine, it needs to be converted into bytecode by a compiler.

**Interpretation**: Interpreted languages, like Javascript, are supposedly not compiled, but simply "executed". However, there are a few misconceptions to clarify:
- Javascript is actually compiled, just not ahead of time into a binary like a traditional compiled language. Most browsers use a JIT (Just-In-Time compiler) to compile the language just as its about to run.
- A JS program is executed in two phases. First, the interpreter reads the entire code (parsing), which is when hoisting occurs. Only then is the code actually executed.

**Hoisting**: Hoisting is a JavaScript mechanism where variables and function declarations are moved to the top of their scope before code execution. Note that only variable declarations are moved, but not assignment of values.

Because of this, in the following code, during the first phase, `a` is initialized to `undefined` and then it can actually run:

```js
console.log(a); // undefined
var a;
```

However, even though `let` and `const` variables are also hoisted (i.e. their declaration moved to the top), they are not *initialized*. 

```js
console.log(a); // ReferenceError: Cannot access 'a' before initialization
let a;
```

Note how the error is NOT telling us that `a` does't exist, but rather that it's not *initialized*.