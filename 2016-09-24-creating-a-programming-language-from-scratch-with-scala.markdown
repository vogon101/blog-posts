---
layout: post
title: "Designing a Programming Language from Scratch"
date: 2016-09-24 19:23:28 +0200
comments: true
categories:
 - open-source
 - Scala
 - programming
 - projects
 - plfs
---

  It has always been a dream of mine to create my own programming language, however useless. I have tried this a few times (see [Slang](https://github.com/vogon101/SLang) and [Slang2](https://github.com/vogon101/SLang)) however these aren't really any good at all. The first one had no type system and was purely procedural and the second had some types but they are badly implemented and don't really work. With this experience behind me, I have decided to start again, from the ground up. This time a few things will be different:

<!-- more -->

 * I will be writing my own parsing code and not using the Scala parser combinators
 * I will design the language around the type system and not the other way round
   * This *should* prevent me getting stuck somewhere down the road with a malfunctioning type system that needs to be completely rewritten

This language will be for embedding in JVM applications, imagine LUA but inspired by java or Scala. This could be used for games (ie mods), an office suite (ie macros) or any other application that needs to allow the user to access a limited set of functions and needs to allow them to control these programmatically. I am using Scala because I love the language and it gives me interoperability with Java.

## The language basics

The language will draw inspiration from a number of sources.

At a high level, the language will have the following principles:

 * It will be interpreted, not compiled
 * The type system will be based on Java's syntactically but behind the scenes the implementation will be like Javascript's, prototype based
 * Functions and types will simply be objects, assigned to variables
 * Syntactically the language will be a cross between Java/C#, scala and my own imagination

## An Example

Here is some example code in the new language, obviously the language may change:

```scala
class Person (name, yearOfBirth) {

	this.age = () =>  currentYear - yearOfBirth;

	this.apply = () => {
		return "Person (" + name + ")"
	}

	//Yes I know this is a stupid function but who cares
	this.+ = (years) => {
		return new Person(name, yearOfBirth+years)
	}

	this.+= = (years) => {
		this.yearOfBirth = this.yearOfBirth + years
	}

}

currentYear = 2016
freddie = new Person("Freddie", 2000)

println(freddie.age()) //Prints 4
println(freddie()) //Prints "Person (Freddie)"

freddie.name = "Freddie Poser"
println(freddie()) // Prints "Person (Freddie Poser)"

newFreddie = freddie + 1
println(freddie.yearOfBirth) //Prints 2000
println(newFreddie.yearOfBirth) //Prints 2001

freddie += 2
println(freddie.yearOfBirth) //prints 2002
println(newFreddie.yearOfBirth) //Prints 2001
```

Here we can see a couple of things about the proposed language:

* The body of the class is the constructor, just like in scala
  * Unlike scala however, functions are simply assigned to variable members of `this` object
    * This is a bit like javascript and makes it easier to implemented
  * Arguments to the constructor are defined just after the class name
* Functions can access the scope of the caller (see currentYear)
  * This makes the language easier for beginners
  * This also might be changed later, depending on how I feel because this seems wrong
* All members are public - this is like python or Javascript
  * This is a dynamic language, `private` or `protected` doesn't really make sense in this
* Objects can be called like functions using the apply method
  * This is borrowed from scala
  * It is included because I like it and means we don't have to parse special conditions for list access
    * See Scala's list
* Functions are defined with the `(<arguments>) => <code>` and simply assigned to variables
  * Because they are just objects they can be passed to functions like so: `higherOrderFunction ( (arg) => println(arg) )`
* Semicolons are optional
* Everything is an expression that can return a value
* Functions return the value of their last line if there is no `return` statement

## Final Thoughts
This language is for embedding in the JVM and so is not a full, static, compiled language. It is designed for fast prototyping and to be easy. There are plenty of aspects in this that I would change if this was a real language, but the design decisions above make the implementation easier and make it simpler for the users.

