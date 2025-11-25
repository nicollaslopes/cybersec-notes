# Prototype Pollution

Prototype pollution is a vulnerability that is specific to programming languages with prototype-based inheritance (the most common one being JavaScript). While the bug is well-known for some time now, it lacks practical examples of exploitation.

## Prototype-based inheritance

Let’s create a simple object in JavaScript:

```javascript
let obj = {
  prop1: 1,
  prop2: 2
}
```

The object `obj` contains two properties called `prop1` and `prop2`. We can access the properties via the standard syntax of `obj.prop1` or `obj.prop2`. These properties aren’t the only ones we can access, though, as there are many others, including `toString`, `constructor` or `hasOwnProperty`. How could it be that we can access them if they’re not direct properties of `obj`? The answer is: prototype.

When we try to access a property that doesn’t exist in an object, JS engine tries to look for the property in the prototype of the object. In JavaScript, we can find out what the prototype is, using `Object.getPrototypeOf(obj)` or simply `obj.__proto__`.

im1

Accessing prototype of `obj`

If we’re creating a simple object using the `{ prop: val, ... }` notation, its prototype is going to be set to `Object.prototype`. We can think of `Object.prototype` as the prototype of all objects (this is not actually true, since we can deliberately create an object without a prototype but it’s usually not done).

So when we try to access `obj.toString`, JS engine check if it is a property of `obj` and if it isn’t, then checks if it’s a property of its prototype. If it still wasn’t true, the checks would follow until a prototype is set to `null` (this is called the [prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)).

## How do prototype pollution vulnerabilities arise?

Prototype pollution vulnerabilities typically arise **when a JavaScript function recursively merges an object containing user-controllable properties into an existing object, without first sanitizing the keys**. This can allow an attacker to inject a property with a key like `__proto__`, along with arbitrary nested properties.

Due to the [special meaning of `__proto__`](https://portswigger.net/web-security/prototype-pollution/javascript-prototypes-and-inheritance#accessing-an-object-s-prototype-using-proto) in a JavaScript context, the merge operation may assign the nested properties to the object's [prototype](https://portswigger.net/web-security/prototype-pollution/javascript-prototypes-and-inheritance#what-is-a-prototype-in-javascript) instead of the target object itself. As a result, the attacker can pollute the prototype with properties containing harmful values, which may subsequently be used by the application in a dangerous way.

It's possible to pollute any prototype object, but this most commonly occurs with the [built-in global `Object.prototype`](https://portswigger.net/web-security/prototype-pollution/javascript-prototypes-and-inheritance#the-prototype-chain).

We can change application behavior. The most commonly shown example is the following:

```javascript
if (user.isAdmin) {
   // do something important!
}
```

Imagine that we have a prototype pollution that makes it possible to set `Object.prototype.isAdmin = true`. Then, unless the application explicitly assigned any value, `user.isAdmin` is always true!

img2

`user.isAdmin` is `true`!

In the screenshot above, even though we didn’t set any property on the `user` object, `user.isAdmin` is still `true` because it inherits the property from the prototype.

Missing attributes come from the prototype: 
```javascript
const a = {};
a.__proto__.someFunction = function () {
  console.log("Hello from the prototype!")
};
a.someFunction();
```

```
Hello from the prototype!
```

Setting attributes on a shared prototype:

```javascript
const a = {};
const b = new Object();
a.__proto__.x = 1337;
console.log(b.x);
```

```
1337
```

A prototype pollution attack where a hacker sends a malicious payload to the backend server, and an unsafe merge function recursively merges that payload with a backend object

This code shows how the vulnerability occurs. Let's take a look at `merge` function, the problem is here. When we send a malicious payload, it will merge our data with the expected data.

```javascript
async function updateUser(userId, requestBody) {
  const userData = await db.loadUserData(userId);
  merge(userData, requestBody);

  log("Saving userData " + userData.toString());
  await db.saveUserData(userId, userData);
  return userData;
}

async function getRole(userId) {
  const userPermissions = await db.loadUserPermissions(userId);

  let role = "user";
  if (userPermissions.role) {
    role = userPermissions.role;
  }

  return { role };
}

/**
 * Sets or updates all attributes of the source object on the target object.
 *
 * For example if `target` is {a: 1, b: 2} and `source` is {a: 3, c: 4},
 * after calling this function `target` becomes {a: 3, b: 2, c: 4}.
 */
function merge(target, source) {
  for (const attr in source) {
    if (
      typeof target[attr] === "object" &&
      typeof source[attr] === "object"
    ) {
      merge(target[attr], source[attr])
    } else {
      target[attr] = source[attr]
    }
  }
}
```


## Client-Side Prototype Pollution Payloads

[[client-side-prototype-pollution](https://github.com/BlackFan/client-side-prototype-pollution)](https://github.com/BlackFan/client-side-prototype-pollution#client-side-prototype-pollution)



