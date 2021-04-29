---
layout: post
title: "everscript #1"
date: 2021-04-28
author: andre
tags: all programming everscript
---

hi, me again!

i wanted to talk a little bit about one of my long-time projects. it's called **EverScript**, and it's a little programming language based upon the magnificent work of none other than **Bob Nystrom**.

the idea is simple: it's a dynamically typed, interpreted and object-oriented language, built in **Java**. it is somewhat slow, so there is a bytecode virtual machine chapter, where the author, **Bob**, uses **C** as his language of choice. i opted to go with the first chapter, and add features that make the language useful. sure, i could've completed the second chapter and then add features to the virtual machine, but for ease of prototyping and expandability, the first chapter is great.

however, some of the core "ingredients" are different. the original **lox** implementation treats numbers in general as **Java** `double`'s. **EverScript**, however, treats numbers as different types, for some added type safety. integer numbers are represented as `long`'s, while numbers with decimal cases are represented as `double`'s. there are some conversion methods, that allow the programmer to convert one type to the other.

**EverScript** also introduces the concept of multiple inheritance. there are traits, which cannot be instantiated, but can implement other traits. classes can, therefore, implement multiple traits as well, and inherit every method from all the previous traits. if a method from one trait has the same name as a method from another trait, **EverScript** will present you with a `RuntimeError`. `override` isn't implemented, but it is in the works.

there are also imports. this allows you to split big projects between multiple files, to keep the codebase clean and organized.

there is a small standard library, which is still in its infant stage, but over time will grow into a big collection of utilities.

there are `test` blocks, an idea that i stole from **Zig**. there's also a handy little `assert` function, that paired with `test` blocks, allows you to write a nice test suite for your next big project or library.

there are `static` and `getter` methods. lambda expressions. dictionaries. arrays. exception handling and `throw`. enumerators.

it is safe to say that **EverScript** took on a different path, to improve on an already great base for a hobby language.

anyways, that is it for today. hope you enjoyed reading a little bit about my implementation of **lox**. if you're feeling curious and adventurous, the github repository for **EverScript** is [here](https://github.com/EverScriptLang/EverScript). 

have a great one, be safe out there! &#10084;