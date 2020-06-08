---
layout: post
title: "Multimethods"
---

Multimethods (or multiple dispatch) is something that sort of completes the picture of object-oriented method calling.
It's not something which is found in many modern programming languages, and it's interesting to look at.

It's a type of polymorphism. Wikipedia defines polymorphism as "the provision of a single interface to entities of different types or the use of a single symbol to represent multiple different types." Specifically, multimethods define a method resolution strategy for subtyping, which we would call inheritance in the object oriented world.

## An Example

I'll just write a basic example with 2 classes, which I'll refer to in the later sections:

```scala
class A {
    def f(a: A) = "f of Class A called with object of class A"
    def f(a: B) = "f of class B called with object of class B"
}

class B extends A {
    def f(a: A) = "f of Class B called with object of class A"
    def f(a: B) = "f of class B called with object of class B"
}
```

In the little example above we have classes A and B where B inherits from A. The classes define a method `f` which can be called on the objects of these classes, with objects of class `A` or `B`. The type of dispatch decides which method will be ultimately called.

```scala
val a: A = new B();
println(a.f(a))
```

What should be printed? Turns out it depends on the type of dispatch used, which we discuss in the following section.

## Different types of Dispatch

When a method is called as `a.method(b1, b2, b3, ...)` where type of `a` is `A` and types of the parameters are `B1`, `B2`, `B3`, ...; the type of dispatch resolves which method to call due to the following possibilites:

1. `a`, although referenced by `A`, might actually point to an object of a subclass, which could've overridden the method.
2. `a` could have multiple methods of the same name, as supported by many languages.

The different types of dispatch would choose different resolutions. Let's see what the types are.

### Static Dispatch

In this type of dispatch, the method to be called can be determined at compile time. This is the way it works with normal inheritance in C++. 
Going by the example we defined earlier, the method is looked in the definition of the class of `a` which is `A` i.e. the type of the reference and not the actual type of the object. Similarly, in the class, the function is chosen based on the type of the reference of the argument passed, which can be determined at compile time. So, the code prints out `f of class A called with object of class A`

### Dynamic Dispatch

Here, one thing is changed from static dispatch, that the actual type of `a` is used to determine which class' method will be called. In the example, the actual class of `a` is `B` which can't always be determined at compile time[^1], and since the type of the reference `a` is `A`, the code prints out `f of class B called with object of class A`. This type of dispatch is used in Java and is supported in C++ with the help of virtual functions.

### Multiple Dispatch (aka Multimethods)

Notice in dynamic dispatch, the object we're calling the method on was treated specially, in that it's type was taken as the type of the actual object, and not the reference. In multiple dispatch, that treatment is extended to the method parameters so the type of the actual object they're pointing to is considered in choosing the method to call. In the example, the type of the actual object referred to by `a` is `B`, so out code ends up printing `f of class B called with object of class B`. This type of dispatch has support in C# with the use of the keyword `dynamic` and has built-in support in Julia.

## Use-case for Multimethods

Multimethods can be useful in binary operators that have been defined in the superclass and overridden by the child classes. For example, let's consider the following class:

```scala
class Shape {
    def collidesWith(s: Shape): Boolean;
}
```

Here, `collidesWith` is an operator that takes 2 shapes, and returns whether they collide (intersect) or not. Since the Shape class doesn't have enough information about the underlying shape, the operator has to be implemented by the child classes such as `Circle`, `Rectangle`, etc. Each of the classes has to have multiple methods taking different subclasses `Shape`.

Then, suppose we have 2 `Shape` references `s1` and `s2`, and want to know whether they collide or not. When we call `s1.collidesWith(s2)`, the method of the appropriate subclass of Shape is called and the appropriate method is also chosen.

Note that this would not have been possible with just dynamic dispatch since no method which takes a `Shape` object would be present. In languages with no support for multiple dispatch, you'll have to define:

```scala
def collidesWith(s: Shape) = s.collidesWith(this)
```

So, the method is chosen in the actual class of `s` (due to dynamic dispatch) and the appropriate method is chosen since `this` has the reference type as a subclass of `Shape`.

This workaround is still fairly easy, and it becomes quite messy in dynamically typed Object Oriented languages (such as Ruby) in which you don't have function overloading. In those languages you'll have to use something like `s.collidesWithCircle(this)` in the `collidesWith` method of the `Circle` class, and so on.

## Conclusion

Multimethods are not present in most languages, but I still find it's good to know about them. It makes one realize that in dynamic dispatch how one thing is treated specially in a method call (the object on which the method is called) and see what possible benefits there could be if that treatment was to be extended to other parameters.

## Footnotes

[^1]: In `a: A = if (condition) new A else new B`, we can't know the class of the actual object `a` is referring to at compile time, since the condition of the if statement might not be known at compile time.
