---
title: Get rid of Null in your Java code and experience the power of Optionals
date: "2019-10-09T15:52:00.000Z"
description: "In Java 8, Optionals were introduced. They can help you getting rid of null values, null checks, and a whole class of possible bugs."
---

In Java 8, Optionals were introduced. They can help you getting rid of Null values, Null checks, and a whole class of possible bugs.

This is not a documentation of all the features and use cases of Optionals. Not at all! It's merely an overview of when and how I use them in my daily life, and why I think they're great.

## Why not to use null?

Null values are a great way to introduce bugs to your code. You can never be sure, if some method does **always** return some valid Object, or if it may return Null. Even if you argue, that this is **your** code, and you know that a method does not return Null: You can never be sure, if this will always be the case. Maybe in a distant future, someone (which may be you) changes a method to return Null in some edge cases. And suddenly, you're prone to `NullPointerException`s.

Another reason to not use Nulls is, that often times its meaning is not obvious. Is it Null indicating an error state, after something bad happened? Is it a bug that Null got returned here? Or is it a totally valid value, indicating that there is nothing to return, which is okay, and you're good go on?

That's why it's good to make things explicit! And in this case, this is done using Optionals.


## What are Optionals?

You can imagine an Optional as a list. A restriction to this "list" is, that it can only contain up to one element. That means, either there is a value, or there is none. [The Java API documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) describes Optionals as

> A container object which may or may not contain a non-null value.
> <cite>Optional (Java Platform SE 8)</cite>

That's kind of everything important to say about it. By wrapping a value, which may or may not be present, into an Optional container, we make it explicit that maybe there will no value be present.


## How do I create Optionals?

To create an Optional, you can use one of its three static factory methods:

```java
String someString = "abc";
Optional<String> stringOpt = Optional.of(someString);
```

`Optional::of` is the default way to wrap some value into an Optional. In your Null-free world, you are sure that the value you'll put into that Optional is not Null. If the value _is_ null, this method will throw a `NullPointerException`, which indicates an unexpected bug in your code that you should fix.

```java
String someStringOrNull = random.nextBoolean() ? "abc" : null;
Optional<String> stringOpt = Optional.ofNullable(someStringOrNull);
```

`Optional::ofNullable` is almost the same as `Optional::of`, except that it will create an empty Optional when you put Null into it. No Exception will be thrown. This is a great way, if you

1. use external libraries that may return Null in some cases.
2. gradually want to get rid of Nulls in your code base, and still have some Null-producing methods left.

```java
Optional<String> stringOpt = Optional.empty();
```

`Optional::empty` does not take any parameters, because is does not contain a value. That is the replacement of your Nulls.

## How do I get the Optional's content?

To retrieve the actual value from the Optional, there are some ways to go. The most obvious would be to call `get()`. But behold! If you call `get()` on an empty Optional, a `NoSuchElementException` gets thrown and you'll have the same problems like before when you used Null. So, you should check for a value to be present inside the Optional, and that's what `isPresent()` is for.

> But how is
> ```java
> if (opt.isPresent()) opt.get().doSomething();
> ```
> better than
> ```java
> if (opt != null) opt.doSomething();
> ```
> ?
> <cite>Someone, Maybe</cite>

Doing things that way is better than having to check for Null, because it's more explicit and obvious that something has to be done here. The `get()` can serve as a reminder that some checking has to be done beforehand. But, yes, _Someone, Maybe_ is right. This is not very beautiful. A look into the documentation reveals more ways to retrieve the Optional's content.

`void ifPresent(Consumer<? super T> consumer)` is another way of doing things with the Optional's content if it's present. Instead of the `isPresent()`-`get()` shenanigans, you could do:

```java
opt.ifPresent(value => value.doSomething());
```

`ifPresent` looks kind of "functional", but it isn't. The `consumer` parameter can't return any value, it probably does have side effects, which is not functional at all. To perform actions on the Optional's content a more functional way, there are some typical "functional" methods defined:

```java
String transformedContent = stringOpt
  .filter(str => !str.isEmpty())
  .map(String::toLowerCase)
  .orElse("some default value");
```

Utilizing these transformative methods to Optionals, you can use them in a strict functional manner. This can lead to a more readable, mainainable and bug-free code.


## How do I replace Nulls by Optionals?

The easiest way to not have Nulls in your code base is to never use Nulls from the beginning. When you are writing a method, and there's a case where you want to return Null, change this method's signature. Instead of returning a type, return an Optional of that type instead.

So, when you want to grab a pizza slice from a pizza box, but you're not sure if there is some pizza left, you could do it like that:

```java
public PizzaSlice grabASlice(PizzaBox box) {
  if (box.containsPizza()) {
    return box.getPizza().getSlice();
  }
  return null;
}
```

Now you, the person looking forward to a hot and cheesy slice of pizza, have to check if you really got a slice in your hand after you retrieved it from the box. However, in your huge anticipation of a greasy slice of your favourite disk-shape food, you're very likely to forget checking. Bite marks in your hand, disappointment (also known as `NullPointerException`s) are the consequences:

```java
PizzaSlice slice = grabASlice(emptyBox);
slice.takeABite(); // bite marks, pain aka NullPointerException
```

That's why it's important to manage expectations beforehand:

```java
public Optional<PizzaSlice> grabASlice(PizzaBox box) {
  if (box.containsPizza()) {
    return Optional.of(box.getPizza().getSlice());
  }
  return Optional.empty();
}
```

Now, you know that the box may be empty before you reach out to get a slice. That's why you _will_ be more alert and check for it before you force your teeth into whatever is in your hand:

```java
Optional<PizzaSlice> slice = grabASlice(emptyBox);
if (slice.isPresent()) {
  slice.get().takeABite();
} // else, disappointment but no bite marks
```

The `.isPresent()` check followed by `.get()` is not the nicest way to unpack things here. A bit more descriptive is the following call, which does the exact same thing.

```java
grabASlice(emptyBox).ifPresent(PizzaSlice::takeABite);
```

## Conclusion

There really is no reason not to using Optionals and getting rid of Nulls in your Java code. I consider methods, that sometimes return Null, as a thing of the past! Java has evolved and is still evolving. You should not "go with the flow" blindly, because new is not always better. Question changes and ask yourself, if some renewal really improves your code. ... But in this case: Nulls are bad. Don't use them. Like, never. Just don't. They bring pain and disappointment.