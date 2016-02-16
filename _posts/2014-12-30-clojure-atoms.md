---
layout: post
cover: 'assets/images/cover6.jpg'
title: Clojure Atoms
date:   2014-12-30 10:18:00
tags: clojure
subclass: 'post tag-programming'
categories: 'mrknmc'
navigation: True
---

For my honours project I need to program in Clojure, a dialect of Lisp that targets the Java virtual machine. I wrote this blog post mainly because writing about something forces me to think about it more and understand it better.

Clojure data structures are immutable. This is so that programs written in Clojure are more functional and hence more robust. To provide mutable state, Clojure provides four reference types: vars, refs, agents, and atoms. Every type is useful in a different scenario but this post focuses on atoms.

### Definition

The Clojure.org [website](http://clojure.org/atoms) describes atoms as follows:

> Atoms provide a way to manage shared, synchronous, independent state.

Let’s break this statement down.

#### Shared state

In this context, shared state means sharing some state between two or more concurrent processes. For example, we could have two threads accessing a value in the same memory location, one trying to update the value and the other trying to read the value.

In most languages, we would need to take care to avoid [race conditions](https://en.wikipedia.org/wiki/Race_condition). With atoms, however, this is handled for the programmer automatically using the [compare-and-swap](http://en.wikipedia.org/wiki/Compare-and-swap) instruction. Thus, changes to atoms are always free of race conditions.

#### Synchronous state

Atoms operate in a synchronous manner. That is if we de-reference an atom, we get its value immediately as opposed to some time in the future when the value becomes “known”. Moreover, if we try to update the value of an atom with an operation that takes e.g. 5 seconds to complete, the thread will block until the operation is completed.

#### Independent state

Even though atoms are shared by multiple threads, the state they provide is independent. This means that changing the value of an atom does not and should not affect some other state.

A typical scenario where this is **_not_** true is moving money between bank accounts. If a bank needs to move some money from account A to account B it either wants to transfer all of it or none of it. It doesn’t want to leave the accounts in some inconsistent state, for example withdrawing money from A without ever depositing it in B. In this scenario we would want to use [refs](http://clojure.org/refs) which are created specifically for coordinated access.

### Short Example

A good example of where we could use an atom is a simple counter.

#### Creating an atom

Creating an atom is easy, we simply use the `atom` function:

```
user=> (def counter (atom 0))
#’user/counter
user=> counter
#<Atom@1cdca0c8: 0>
```

Here we defined an atom called `counter` in namespace `user`. Its value is 0.

#### Reading an atom

To read the value of an atom we use the `@` reader macro or the `deref` function:

```
user=> @counter
0
user=> (deref counter)
0
```

#### Updating an atom

To update the value associated with the atom we use the `swap!` function:

```
user=> @counter
0
user=> (swap! counter inc)
1
user=> @counter
1
```

Function `swap!` is defined as:

```
(swap! atom f args)
```

It atomically swaps the value of the atom to be:

```
(apply inc current-value-of-atom args)
```

In our case `args` is empty and thus we got 1 as a result:

```
user=> (apply inc @counter [])
1
```

Since the value of the atom could have been changed by some other thread before it was swapped by the current thread `swap!` may need to be retried until successful and thus the function that it applies has to be free of side effects. In our case this is true since 0 + 1 is always equal to 1.

As you can see, atoms can be very powerful even though working with them is quite simple. They are also fast:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">A Clojure map inside an atom is about 2x more efficient than a HashMap with a lock <a href="http://t.co/QnkUzWo">http://t.co/QnkUzWo</a></p>&mdash; Nathan Marz (@nathanmarz) <a href="https://twitter.com/nathanmarz/status/106051181226364928">August 23, 2011</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

---

##### Sources

- [http://clojure.org/atoms](http://clojure.org/atoms)
- [http://clojure-doc.org/articles/language/concurrency_and_parallelism.html](http://clojure-doc.org/articles/language/concurrency_and_parallelism.html)
