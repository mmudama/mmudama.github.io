+++ 
draft = false
date = 2025-08-21T17:44:24-06:00
title = "Fun With Golang Slices"
description = "Slices will surprise you"
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

When I first encountered Go, I found [slices](https://golangdocs.com/slices-in-golang) puzzling. You can create slices from an array, and the slice will be a pointer to a portion of the array. Got it. But wait, you can also append to that slice - how does that even work? The [Cambridge Dictionary](https://dictionary.cambridge.org/dictionary/english/append) defines "append" as: "to add something to the end of a piece of writing." Code, writing, same thing. But if you're appending something to a slice, what's happening to the original array? If it's modified, does that mean you're appending to a slice while overwriting a value in the array? If so, then what happens when you append past the end of the array? How would THAT work?

The answer, it turns out, is a bit complicated. 


Okay, let's start with an array!
```golang
startingArray := [3]int{1, 2, 3}
```
Next, we'll create two slices. They will each be references pointing into a subset of the array, and they'll both contain that middle value, 2.

```golang
headSlice := startingArray[0:2]
tailSlice := startingArray[1:3]
```

At this point I'll mention that both arrays and slices have lengths and capacities. This will be relevant in a bit.

| Variable       | Length | Capacity |
|----------------|--------|----------|
| startingArray  | 3      | 3        |
| headSlice      | 2      | 3        |
| tailSlice      | 2      | 2        |


Now we have 
```golang
startingArray = [1 2 3]
headSlice = [1 2]
tailSlice = [2 3]
```

So far, so good. Everything works as expected.

Then we do this ...
```golang
headSlice = append(headSlice, 4)
```

And get this ...

```golang
startingArray = [1 2 4]
headSlice = [1 2 4]
tailSlice = [2 4]
```

I don't know how to feel about this. It seems like a pretty big side effect; a side effect that the function name doesn't convey. My intuition from years in other languages is that this is Wrong. But maybe it's just a matter of the [Two Hard Things](https://martinfowler.com/bliki/TwoHardThings.html) problem; they had to call it *something*. The slices reference the array, so if one changes, they all have to change. Except ... wait ... what happens when I append past the end of the array? Do I get a runtime error? Do I walk right off the end of the array, like we're writing code in C? Or ... does the behavior change completely? And this is where my eyelid really starts to twitch.

```golang
headSlice = append(headSlice, 5)
startingArray[2] = 999
```
results in ...

```golang
headSlice = [1 2 4 5]
startingArray = [1 2 999]
```

Closer inspection (see the full code linked at the end of the post) reveals that `headSlice` is no longer pointing to `startingArray`, and its capacity has changed. Before this, a change to `headSlice` would change `startingArray`, and vice versa. After this, they have parted ways - they can't affect each other anymore.

| Variable       | Length | Capacity |
|----------------|--------|----------|
| startingArray  | 3      | 3        |
| headSlice      | 4      | 6        |



So, that's how Go handles appending when it would result in a buffer overrun of the backing array: it creates a new array with more capacity and points to that instead.

This wouldn't bother me so much if there were a flag to indicate what's happening. As far as I know, there is not. You can, however, check whether an append will grow the length of the slice beyond its capacity, in which case you know that an append will create a new backing array:

```golang
if cap(headSlice) == len(headSlice) {
    fmt.Printf("Head Slice will create a new backing array on append; append WILL NOT affect the array\n")
} else {
    fmt.Printf("Head Slice can still grow; append WILL affect the array\n")
}
 ```

If you use that conditional, make sure the code is [thread safe](https://en.wikipedia.org/wiki/Thread_safety)! 

 Why does any of this matter? Well, in every language, you need to understand whether you're working with reference or value types. Golang slices are a reference type - they refer to another value. Golang arrays are a value type, which isn't terribly common. (Looks like they're also value types in Rust.) [Here's](https://en.wikipedia.org/wiki/Value_type_and_reference_type) a reference on value vs reference types.

The idea that an append can quietly choose to either modify or copy the backing data, even though that data can be accessed through other variables, is profoundly weird to me. Every language has its gotchas. As a newbie coder, I learned to use a pointer and a length to pass around strings and arrays in C and C++; an off-by-one error meant clobbering the computer's memory. Debugging that code was so fun! Wait, I mean miserable. Debugging that code was miserable. Anyway, a reasonable person could go about using slices in Go for quite a while without ever noticing this behavior - until suddenly one day, things got real weird. I hope I've saved you from that fateful day.

By the way, you get this same behavior even if you start with a slice instead of an array. A slice is always backed by an array, even if you didn't create the array explicitly. You can replace `[3]int{1, 2, 3}` with `[]int{1, 2, 3}` and get the same results.

Thanks for reading.

[Full Code](https://github.com/mmudama/misc/blob/master/playground/sliced-arrays/sliced-arrays.go)