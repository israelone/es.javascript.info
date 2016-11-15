# F.prototype

In modern Javascript we can set a prototype using `__proto__`. But it wasn't like that all the time.

[cut]

JavaScript has had prototypal inheritance from the beginning. It was one of the core features of the language.

But in the old times, there was another (and the only) way to set it: to use a `"prototype"` property of the constructor function. And there are still many scripts that use it.

## The "prototype" property

As we know already, `new F()` creates a new object. But what we didn't use yet `F.prototype` property.

That property is used by the Javascript itself to set `[[Prototype]]` for new objects.

**When a new object is created with `new F()`, the `[[Prototype]]` of it is set to `F.prototype`.**

Please note that `F.prototype` here means a regular property named `"prototype"` on `F`. It sounds something similar to the term "prototype", but here we really mean a regular property with this name.

Here's the example:

```js run
let animal = {
  eats: true
};

function Rabbit(name) {
  this.name = name;
}

*!*
Rabbit.prototype = animal;
*/!*

let rabbit = new Rabbit("White Rabbit"); //  rabbit.__proto__ == animal

alert( rabbit.eats ); // true
```

Setting `Rabbit.prototype = animal` literally states the following: "When a `new Rabbit` is created, assign its `[[Prototype]]` to `animal`".

That's the resulting picture:

![](proto-constructor-animal-rabbit.png)

On the picture, `"prototype"` is a horizontal arrow, it's a regular property, and `[[Prototype]]` is vertical, meaning the inheritance of `rabbit` from `animal`.

## Summary

In this chapter we briefly described the way of setting a `[[Prototype]]` for objects created via a constructor function. Later we'll see more advanced programming patterns that rely on it.

Everything is quite simple, just few notes to make things clear:

- The `F.prototype` property is not the same as `[[Prototype]]`.
- The only thing `F.prototype` does: it sets `[[Prototype]]` of new objects when `new F()` is called.
- The value of `F.prototype` should be either an object or null: other values won't work.
-  The `"prototype"` property only has such a special effect when is set to a constructor function, and it is invoked with `new`.

On regular objects this property does nothing. That's an ordinary property:
```js
let user = {
  name: "John",
  prototype: "Bla-bla" // no magic at all
};
```