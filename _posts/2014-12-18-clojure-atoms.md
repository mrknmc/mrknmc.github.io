---
layout: post
published: true
title: Clojure Atoms
description: "An introduction to Clojure Atoms."
modified: 2014-12-30
tags: [clojure, atoms]
comments: true
image:
  feature: clojure-atoms.jpg
  credit: Giorgio Galeotti
  creditlink: https://flic.kr/p/asy5XU
---

For my honours project I need to program in Clojure, a Lisp-like programming language that targets the Java virtual machine. I wrote this blog post mainly because writing about something forces me to think about it more and understand it better.

Data structures in Clojure are immutable. This is so that programs written in Clojure are more functional and hence more robust. To provide mutable state, Clojure provides four reference types: [vars](http://clojure.org/vars), [refs](http://clojure.org/refs), [agents](http://clojure.org/agents), and [atoms](http://clojure.org/atoms). In this post I will only talk about atoms.

## Definition

The Clojure.org website describes atoms as follows: [1]

> Atoms provide a way to manage shared, synchronous, independent state.

Let's break this statement down.

### Shared state

In this context, shared state means sharing some state between two or more concurrent processes. For example, we could have two threads accessing the same memory location, one trying to update its value and other trying to read its value. In most languages, we would need to take care to avoid [race conditions](https://en.wikipedia.org/wiki/Race_condition). With atoms, however, this is handled for the programmer automatically using the [compare-and-swap](http://en.wikipedia.org/wiki/Compare-and-swap) instruction. Thus, changes to atoms are always free of race conditions.

### Synchronous state

Atoms operate in a synchronous manner. That is if we de-reference an atom, we get its value immediately as opposed to some time in the future when the value becomes "known". Moreover, if we try to update the value of an atom with an operation that takes e.g. 5 seconds to complete, the thread will block until the operation is completed.

### Independent state

Even though atoms are shared by multiple threads, the state they provide is independent. This means that changing the value of an atom does not affect some other state.

A typical scenario where this is _not_ true is moving money between bank accounts. If a bank needs to move some money from account A to account B it either wants to transfer all of it or none of it. It doesn't want to leave the accounts in some inconsistent state, for example withdrawing money from A without depositing it in B. In this scenario we would want to use [refs](http://clojure.org/refs) which are for coordinated access.

A good example of where we could use an atom is a counter.

## Creating an atom

Creating an atom is easy, we simply use the `atom` function:

{% highlight clojure %}
user=> (def counter (atom 0))
#'user/counter
user=> counter
#<Atom@1cdca0c8: 0>
{% endhighlight %}

Here we defined an atom called `counter` in namespace `user`. Its value is 0.

### Reading an atom

To read the value of an atom we use the `@` reader macro or the `deref` function:

{% highlight clojure %}
user=> @counter
0
user=> (deref counter)
0
{% endhighlight %}

### Updating an atom

To update the value associated with the atom we use the `swap!` function:

{% highlight clojure %}
user=> @counter
0
user=> (swap! counter inc)
1
user=> @counter
1
{% endhighlight %}

Function `swap!` is defined as `(swap! atom f args)` and it atomically swaps the value of the atom to be `(apply inc current-value-of-atom args)`. In our case `args` is empty and thus we got 1 as a result:

{% highlight clojure %}
user=> (apply inc @counter [])
1
{% endhighlight %}

Since the value of the atom could have been changed by some other thread `swap!` may need to be retried and thus the function that is applied has to be free of side effects. In our case this is true since `(inc 0)` is always equal to 1.

As you can see, atoms can be very powerful even though working with them is quite simple. They are also fast:

https://twitter.com/nathanmarz/status/106051181226364928

<!-- ## Notes -->

[1]: http://clojure.org/atoms
[2]: http://clojure-doc.org/articles/language/concurrency_and_parallelism.html
