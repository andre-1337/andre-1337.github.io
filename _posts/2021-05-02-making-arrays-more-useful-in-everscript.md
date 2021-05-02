---
layout: post
title: "making arrays more useful in everscript"
date: 2021-05-02
author: andre
tags: all programming everscript
---

at last!

in my last post, i talked a little bit about my programming language, **EverScript**. it is a shameless "clone" of **JavaScript**, albeit much slower, and with less features (or at least ones that one would deem as "useful").

today, we are making arrays more useful, by adding small utility methods such as `.forEach()` and `.map()`. if you are working on your own **lox** implementation in **Java**, you could easily adapt this to your own implementation!

with much further ado, lets get started!

arrays in **EverScript** are represented by the `ESArray` class (or as i like to call them, objects) in the **Java** side. this handles the runtime part of arrays, such as adding methods to them.

the `ESArray` object implements a little interface, `ESIndexable`. this interface provides 3 methods that we need to override in our `ESArray` object: `get`, `set` and `length`.

```java
interface ESIndexable {
  Object get(Token token, Object index);
  void set(Token token, Object index, Object item);
  int length();
}
```

i won't get into much too much detail about the `ESArray` object, since it is a somewhat long bit of code (over 100 lines!!). if you want to check out the old implementation, you can check [this](https://github.com/EverScriptLang/EverScript/blob/master/src/com/linkbyte/everscript/ESArray.java).
the old implementation was slightly bigger without the utility methods that we are about to add, but most of the length was due to the additional checks that we needed to convert `double`s to the closest `int` number. since **EverScript** version 2 introduces some breaking changes related to how numbers are handled, the code needed to make this conversion is much smaller!

anyways, back to the main topic.

so, how does `.forEach()` work, you may ask? according to the **Mozilla** documentation for **JavaScript**, the `.forEach()` method executes a given function (also known as a callback) once per array element.

although we could've implemented `.forEach()` in the core library of **EverScript**, it would not be as performant as the **Java** counterpart. so, how do we achieve this in java? easy.

first, we need to obtain the size of the array:

```java
@Override public int length() {
  return elements.size();
}
```

remember, since our `ESArray` object implements the `ESIndexable` interface, we need to use the `Override` decorator. with this, we can now access `<array>.length()`. our `ESArray` object has a method that is called by the constructor and goes by the name of `createMethods`. it takes an `ESArray` object as its argument (in this case, `this`).

since **EverScript** has lambda functions and other stuff that **lox** does not offer, your mileage may vary in the next bit of code.

we start by declaring the `.forEach()` method:

```java
methods.put("forEach", new ESCallable() {
  // ...
});
```

in this bit of code, we are adding the `.forEach()` method to the `ESArray` object. this will allow us to do stuff like `[].forEach(function() => ;);` in our **EverScript** code. pretty neat, right? we make use of the `ESCallable` object to define a function/method that can be invoked from within **EverScript**.

`ESCallable` itself is an interface as well, and provides a group of methods that we must `@Override`.

so, we replace the `// ...` line with the following:

```java
  @Override public int arity() {
    return 1;
  }

  @Override public Object call(Interpreter interpreter, List<Object> arguments, Token caller) {
    // ...
  }
```

here, we specify that `.forEach()` takes a single argument (our callback). the `call()` method is where we write the code that will add the functionality to make `.forEach()` work as intended.

`call()` takes 3 arguments: the `Interpreter` instance, a `List` of arguments, by the type of `Object`, and the token of the caller. this is all handled in the `Interpreter`. the caller is simply the token of the parenthesis that closes a call expression. this is used for better error detection and handling.

inside the `call()` method, we add some checks:

```java
  ESFunction callback;
  if (!(arguments.get(0) instanceof ESFunction)) {
    throw new RuntimeError(caller, "Expected a callback.");
  } else callback = (ESFunction) arguments.get(0);

  if (callback.arity() < 1 || callback.arity() > 1) {
    throw new RuntimeError(
      caller, "Expected callback to take 1 argument, got " + callback.arity() + " instead."
    );
  }
```

what do these checks do? well, in the first one, we need to check if the argument provided to `.forEach()` is a function. if this check fails, we raise a `RuntimeError`. otherwise, we cast the argument to our `ESFunction` object and assign it to the `callback` variable.

the second check makes sure that the callback takes a single parameter. if it fails, we raise another `RuntimeError`, specifying that the callback can only take a single parameter.

now, this is where the magic happens:

```java
  for (int i = 0; i < array.length(); i++) {
    callback.call(
      interpreter, 
      new ArrayList<>(Collections.singletonList(array.elements.get(i))), 
      caller
    );
  }
```

with these 3 lines of code, we iterate through all the values in the array, and call the callback while providing the next element. the code may not be the prettiest, but it gets the job done!

we make use of `Collections.singletonList()` since it creates a new list with a single value, which is all we need to pass to the next call. this loop keeps going until our variable `i` is less than the size of the array.

finally, we finish this method by returning `null`:

```java
  return null;
```

and this is it for `.forEach()`. if your implementation also makes use of lambdas, you can test the following to make sure that this method works as intended:

```javascript
let array = [ 1, 2, 3 ];
array.forEach(function(elem) => println(elem);); // [ 1, 2, 3 ]
```

this will print the 3 numbers in the array with newlines on the console.

now, lets get to `.map()`. according to the **Mozilla** documentation for **JavaScript**, `.map()` creates a new array with the results of calling the provided function (or as we know it, callback) on every element of the array.

this may sound a little bit confusing, so lets take it down a notch:
- it takes in a callback as the single argument
- it applies the callback to every single element of the array
- it keeps track of the results of each loop
- it returns a new array with those results

how do we implement this? should be easy enough.

first of all, we start by declaring the `.map()` method:

```java
methods.put("map", new ESCallable() {
  // ...
});
```

the easy part is done. again, we re-use the checks from `.forEach()`, since `.map` only takes a callback as an argument and the callback only takes a single argument as well. so, we replace `// ...` with the following:

```java
  @Override public int arity() {
    return 1;
  }

  @Override public Object call(Interpreter interpreter, List<Object> arguments, Token caller) {
    ESFunction callback;
    if (!(arguments.get(0) instanceof ESFunction)) {
      throw new RuntimeError(caller, "Expected a callback.");
    } else callback = (ESFunction) arguments.get(0);

    if (callback.arity() < 1 || callback.arity() > 1) {
      throw new RuntimeError(
        caller, "Expected callback to take 1 argument, got " + callback.arity() + " instead."
      );
    }
    // ...
  }
```

again, `call` is where all the magic happens. i don't need to go over all the checks again, since they are a straight copy paste of the previous ones.

below the last check, we add a declaration for the `List` of `Object`s where we will place all the resulting values.

```java
  List<Object> elements = new ArrayList<>();
```

this is where we will place all the values while we iterate the array. next, we put another loop, similar to the next one:

```java
  for (int i = 0; i < array.length(); i++) {
    // ...
  }
```

this loop iterates the array just like the `.forEach()` function would do. except, we have a couple of extra things to add:

```java
  Object element = callback.call(
    interpreter, 
    new ArrayList<>(Collections.singletonList(array.elements.get(i))), 
    caller
  );
```

here, we store the result of the callback call in the variable, so we can push it onto the array:

```java
  elements.add(element);
```

with this, the loop is complete. but we still need to make the `.map()` method return a new array, containing all the modified values. how do we do that? well, we return a new `ESArray` object, and pass `elements` as the argument. this will also allow us to chain multiple `.map()` calls on top (if you are *slightly* insane), or call `.forEach()` on the resulting array.

```java
  return new ESArray(elements);
```

now, time to take this for a little test drive. using the following snippet of valid **EverScript** code, we can multiply each item of the array by 2:

```javascript
let array = [ 1, 2, 3 ];
println(array); // [ 1, 2, 3 ]
let new_array = array.map(function(elem) => elem * 2;);
println(new_array); // [ 2, 4, 6 ];
```

great, right?! if you want to get a little bit fancier, though:

```javascript
let array = [ 1, 2, 3 ];
array
  .map(function(elem) => elem * 2;)
  .forEach(function(elem) => println(elem);); // [ 2, 4, 6 ];
```

since we return a new `ESArray` object, we can call `.forEach()` directly on the `.map()` method, or just chain `.map()` at your hearts desire.

with this, we just added a bunch of new possibilities to **EverScript**, although you can add this to just about any **lox** implementation, by adapting the code. i'm actually surprised that i was able to add these little utility methods, since i was thinking that it would be considerably harder to add something this simple.

i know this post is considerably bigger than the last one, but hopefully it can compensate for my lack of presence for the past few days.

as always, i love you all &#10084; be safe out there, and don't forget that today is mother's day for some parts of the world. make sure to tell your mom that you love her (if you happen to live in one of the 9 countries that celebrates this occasion today), if you have any contact with her. thank you for taking the time to read this post, and i'll see you next time!