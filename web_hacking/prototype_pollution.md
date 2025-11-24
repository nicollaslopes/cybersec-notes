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

## Prototype pollution

So where’s the prototype pollution? It happens when there’s a bug in the application that makes it possible to overwrite properties of `Object.prototype`. Since every typical object inherits its properties from `Object.prototype`, we can change application behavior. The most commonly shown example is the following:

```javascript
if (user.isAdmin) {
   // do something important!
}
```

Imagine that we have a prototype pollution that makes it possible to set `Object.prototype.isAdmin = true`. Then, unless the application explicitly assigned any value, `user.isAdmin` is always true!

img2

`user.isAdmin` is `true`!

In the screenshot above, even though we didn’t set any property on the `user` object, `user.isAdmin` is still `true` because it inherits the property from the prototype.

