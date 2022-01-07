---
layout: post
date: 2019-03-02
title: Shooting Yourself in the Foot While Adding an Element to a Kotlin List
description: The functions `add()` and `plus()` which are available on some Kotlin collections, and how despite their similar looking names, their underlying implementations can make a big difference.
keywords:
  - kotlin
  - add
  - plus
  - adding to kotlin collection
  - should I use add or plus to add to kotlin list
  - operator overloading
  - performance
  - arraylist

permalink: 2019/03/shooting-yourself-in-the-foot-while-adding-an-element-to-a-kotlin-list/
---

*This post discusses the functions `add()` and `plus()` which are available on some Kotlin collections, and how despite their similar looking names, their underlying implementations can make a big difference*

## A primer on operator overloads

Kotlin supports [Operator Overloading](https://kotlinlang.org/docs/reference/operator-overloading.html), which means you can make use of operators like `+` and `+=` as a shorthand for calling otherwise normal functions. However, it's always worth checking what the underlying function actually does as otherwise you might catch yourself out. 

For example, to add an `Int` to a collection, there are a few options available, and you'd be forgiven for thinking they are all syntactic sugar for the same functionality.

```kotlin

list + 3 
list.plus(3)
list += 3
list.add(3) 

// note, depending on the collection type, some of these will compile and some won't

```

Consider the following code:
```kotlin

val foo = emptyList<Int>()
foo + 1
foo + 2
printlist(foo)

```

You might expect this to result in a list which contains 2 items `[1, 2]`. 

Except it won't. You'll be left with an empty list.
    
    // OUTPUT
    0 items in list: []

To see why, we can explore the function that is invoked when you use the overloaded `+` operator on a `List`.

```kotlin

/**
 * Returns a list containing all elements of the original collection and then the given [element].
 */
public operator fun <T> Collection<T>.plus(element: T): List<T> {
    val result = ArrayList<T>(size + 1)
    result.addAll(this)
    result.add(element)
    return result
} 
```

Note the signature of this function: it has `operator` in the signature and is called `plus` - this is why it is invoked when you call `foo + 1`.

What does this function do?
    
1. creates a new list
2. adds all elements from the original list to it
3. adds the new element to it
4. returns the new list. 

Here's the important thing to remember: **it doesn't modify the original list**. So this now explains why, in our original example, we were left looking at an empty list. When we call `foo + 1` we are given a new list back that we ignore. We call `foo + 2` and are given another new list back, which we ignore again. 

To fix this, we need to stop ignoring the returned list. One way we can do that is to reassign the original list var to the new list which is returned each time (note because we are reassigning the variable, we need to switch from `val` to `var`):

```kotlin

var foo = emptyList<Int>()
foo = foo + 1
foo = foo + 2
printlist(foo)

```

`foo = foo + 1` can be condensed, using another overloaded operator `+=`.

```kotlin

var foo = emptyList<Int>()
foo += 1
foo += 2
printlist(foo)

```

    // OUTPUT
    2 items in list: [1, 2]

## Where it gets confusing

```kotlin

val foo = ArrayList<Int>()
foo.plus(1)
foo.add(2)
printlist(foo)

```

What we're left with, in some collections, is the ability to choose between two *very similar* looking methods which do different things.

We now know calling the `.plus()` function is equivalent to calling `+`, and therefore will have the behaviour of returning a brand new list.

What about the `.add()` function? 

```kotlin

/**
* Appends the specified element to the end of this list.
*
* @param e element to be appended to this list
* @return <tt>true</tt> (as specified by {@link Collection#add})
*/
public boolean add(E e) {
  ensureCapacityInternal(size + 1);  // Increments modCount!!
  elementData[size++] = e;
  return true;
}

```

From the source, we can see that `.add()` does not return a new list and **it does modify the original list**. 

My brain sees `plus` and `add` as being almost entirely synonymous and yet their behaviour is entirely different. 😰

## Further confusion

```kotlin

// this doesn't compile
var list = ArrayList<Int>()
list += 1

// this doesn't compile
var list: MutableList<Int> = ArrayList()
list += 1

// this compiles 🤨
var list: List<Int> = ArrayList()
list += 1

```

In the first two examples above, we see a compilation error where we try to add to the list using `+=` operator; `Assignment operators ambiguity`. So the implementation matters less here than the declared type (which isn't surprising, per se); depending on the declared type you'll possibly have multiple places where `+=` could work, and therefore the compiler doesn't know which function you would actually want invoked.

In the third example, this compiles OK as there is only function could be invoked for `+=` usage.

## Even more confusion
```kotlin

// option 1
var list: List<Int> = ArrayList()
list += 1

// option 2
var list: List<Int> = mutableListOf()
list += 1

// option 3
val list = mutableListOf<Int>()
list += 1

```

Under the hood, in each case above we are dealing with `ArrayList`s. But the bevavior of `+=` is different. 

1. Calling `+=` copies the whole list into a new list
2. Calling `+=` copies the whole list into a new list
3. Calling `+=` adds the element to existing list

Why? As above, the type you've declared the list as (rather than the implementation itself) is the determining factor in what `+=` will actually do when you use it.

When the declared type is a `List`, then calling `+=` will result in creating a brand new list and copying the old one into it. This is because a `List` type doesn't indicate it is mutable. Whereas if you explicitly have a `MutableList` then `+=` can do the more performant adding of the item without creating a whole new list.

## So, plus() or add() - which to choose
There's probably not a single answer here; which is best will depend on your use case. For some collection types, there won't even be an option. For other collections, like `ArrayList`, you'll be able to choose either approach.

There are times when ensuring your original list is not mutated is beneficial, and conversely there are times when performance concerns might mean you don't want to create a new copy of the list each time you add to it.

I will say that I think `.add()` is going to outperfom `.plus()` in most, if not all, cases; it's cheaper to expand an `ArrayList` than it is to copy the whole list and add a new entry to it.


### Crude performance test
For consideration, I was curious how both the `.add()` and `.plus()` methods would perform on a large-ish list of `Integers`. 

```kotlin
val itemsToAdd = 500_000

val tMutableList = measureNanoTime {
    val list = mutableListOf<Int>()
    for (i in 1..itemsToAdd) {
        list += i
    }
}
Timber.i("Using += on `MutableList<Int>` took ${TimeUnit.NANOSECONDS.toMillis(tMutableList)}ms")


val t = measureNanoTime {
    var list: List<Int> = mutableListOf()
    for (i in 1..itemsToAdd) {
       list += i
    }
}
Timber.i("Using += on `List<Int>` took ${TimeUnit.NANOSECONDS.toMillis(t)}ms")
```

Want to take a guess how long each took? ⏲️

    Using += on `MutableList<Int>` took 74ms
    Using += on `List<Int>` took 405,390ms (over 6 mins!!)
  
## Round up  
Despite the similiarities when you're speaking English, there are important differences when you're speaking Kotlin.

```kotlin

    list + 3 // you've ignored the new list; here be dragons ⚠️🐉
    list.plus(3) // nope! same as above
    list.add(3) // modifies `list`; resizes and inserts new entry
    list += 3 // 🤷‍♂️ it depends on the declared type of the list

```

## Lessons
  - Never use a overloaded operator without ensuring you understand the function it will invoke
  - The declared type of the variable matters in determining which function will be invoked (the actual implementation is less important)
  - It's easy to shoot yourself in the foot. 👢🔫

## Thanks
_The subtle differences described above were unknown to me until my teammate sent a PR my way which changed from using `+=` to using `.add()`. So thanks, **Mia Alexiou**, for teaching me something new today and for a making a tiny change which is likely to result in a good performance boost for our users._ 👏

_Also, after I posted this article it raised quite a few discussions with different people. So thanks for everyone who discussed it with me, and in particular, thanks to **Carmen Alvarez** and **Said Tahsin Dane** ([@tasomaniac](https://twitter.com/tasomaniac)) for pushing my understanding of these nuanaces further._