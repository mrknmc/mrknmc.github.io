---
layout: post
published: true
title: Clojure Atoms
description: "A simple blog post about atoms in Clojure."
modified: 2014-12-18
tags: [clojure, atoms]
comments: true
image:
  feature: clojure-atoms.jpg
  credit: Giorgio Galeotti
  creditlink: https://flic.kr/p/asy5XU
---

<script src="http://www.sveido.com/mermaid/dist/mermaid.full.min.js"></script>

Data structures in Clojure are immutable. This is so that programs written in Clojure are more functional and hence more robust. To provide mutable state, Clojure provides four reference types: [vars](http://clojure.org/vars), [refs](http://clojure.org/refs), [agents](http://clojure.org/agents), and [atoms](http://clojure.org/atoms). In this post I will focus on atoms.

The Clojure website describes atoms as follows:

> Atoms provide a way to manage shared, synchronous, independent state.

Let's break this down.

### Shared state

In this context, shared state means sharing some state between two or more concurrent processes. For example, we could have two threads accessing the same memory location `x`, one trying to update its value and the other trying to read the value:

<div class="mermaid" style="text-align: center;">
    graph TD;
    classDef default fill:#f9f,stroke:#333;
    A[Process 1];
    B[Process 2];
    C{x = 5};
    A-->|read x|C;
    B-->|set x = 2|C;
</div>

### Synchronous state

Atoms operate in a synchronous manner. That is if we de-reference an atom, we get its value immediately as opposed to some time in the future when the value becomes "known". Moreover, if we try to update the value of an atom with an operation that takes e.g. 5 seconds to complete, the thread will block until the operation is completed.

### Independent state

Even though atoms are shared by multiple threads, the state they provide is independent. This means that changing the value of an atom does not affect some other state.

A typical example where this is not true is moving money between bank accounts. If you are a bank and you need to move some money from a bank account A to a bank account B you either want to transfer all of it or none of it. You definitely don't want to leave the accounts in some inconsistent state, for example withdraw the money from A but not deposit it in B.
<!-- ^ this phrasing is weird -->

A good example of where you could use an atom is a counter.

### Creating an atom

Creating a counter is easy, you simply use the `atom` function:

{% highlight clojure %}

user=> (atom 0)
#<Atom@5b72fb19: 0>

{% endhighlight %}

### Reading an atom

To read the current value associated with  you can either use the `@` reader macro or the `deref` function:

{% highlight clojure %}

user=> @(atom 0)
0
user=> (deref (atom 0))
0

{% endhighlight %}

### Updating an atom

{% highlight clojure %}

user=> (swap! (atom 0) inc)
1

{% endhighlight %}


### Example

{% highlight clojure %}

(def server
    (let [a (blah "str")]
        (momo)))

{% endhighlight %}
