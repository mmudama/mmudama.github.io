+++ 
draft = false
date = 2025-08-21T13:44:24-06:00
title = "Fun With Golang Slices"
description = "Slices will surprise you"
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

When I first encountered Go, I found [slices](https://golangdocs.com/slices-in-golang) puzzling. You can create slices from a starting slice or array, and the new slice will be a pointer to a portion of the original data. Got it. But wait, you can also append to the new slice - how does that even work? If you're appending something to the new slice, what's happening to the original slice? If it's modified, does that mean you're appending to the new slice while overwriting a value in the old one? If so, then what happens when you append past the end of the old slice? How would THAT work?

The answer, it turns out, is a bit complicated. 

Let's start by creating a slice.

```golang
startingSlice := []int{1, 2, 3}
```
Next, we'll create two new slices. They will each be references pointing into a subset of the starting slice, and they'll both contain that middle value, 2.

```golang
headSlice := startingSlice[0:2]
tailSlice := startingSlice[1:3]
```

At this point I'll mention that slices have lengths and capacities. This will be relevant in a bit.

| Variable       | Length | Capacity |
|----------------|--------|----------|
| startingSlice  | 3      | 3        |
| headSlice      | 2      | 3        |
| tailSlice      | 2      | 2        |


Now we have 
```golang
startingSlice = [1 2 3]
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
startingSlice = [1 2 4]
headSlice = [1 2 4]
tailSlice = [2 4]
```

I don't know how to feel about this. It seems like a pretty big side effect; a side effect that the function name doesn't convey. My intuition from years in other languages is that this is Wrong. But maybe it's just a matter of the [Two Hard Things](https://martinfowler.com/bliki/TwoHardThings.html) problem; they had to call it *something*. The head and tail slices reference the starting slice, so if one changes, they all have to change. Except ... wait ... what happens when I append past the end of the starting slice? Do I get a runtime error? Do I walk right off the end of the starting slice? Or ... does the behavior change completely? And this is where my eyelid really starts to twitch.

```golang
headSlice = append(headSlice, 5)
startingSlice[2] = 999
```
results in ...

```golang
headSlice = [1 2 4 5]
startingSlice = [1 2 999]
```

Closer inspection (see the full code linked at the end of the post) reveals that `headSlice` is no longer pointing to `startingSlice`, and its capacity has changed. Before this, a change to `headSlice` would change `startingSlice`, and vice versa. After this, they have parted ways - they can't affect each other anymore.

| Variable       | Length | Capacity |
|----------------|--------|----------|
| startingSlice  | 3      | 3        |
| headSlice      | 4      | 6        |



So, that's how Go handles appending when it would result in a buffer overrun of the backing array: it creates a new backing array with more capacity and points to that instead.

This wouldn't bother me so much if there were a flag to indicate what's happening. As far as I know, there is not. You can, however, check whether an append will grow the length of the slice beyond its capacity, in which case you know that an append will make a copy instead of just referencing the parent slice:

```golang
if cap(headSlice) == len(headSlice) {
    fmt.Printf("Head Slice will make a copy on append; append WILL NOT affect the starting slice\n")
} else {
    fmt.Printf("Head Slice can still grow; append WILL affect the starting slice\n")
}
 ```

If you use that conditional, make sure the code is [thread safe](https://en.wikipedia.org/wiki/Thread_safety)! 

The idea that an append can quietly choose to either modify or copy the backing data, even though that data can be accessed through other variables, is profoundly weird to me. A reasonable person could go about using slices in Go for quite a while without ever noticing this behavior - until suddenly one day it becomes extremely relevant. Usually in the production environment. I hope I've saved you from that fateful day.

By the way, everything I wrote here applies if you start with an array instead of a slice. In fact, that's how I originally wrote the code and the post. But since few people actually work directly with arrays in Golang, using slices seems like a better demonstration.

Thanks for reading.

*This blog post was mentioned on the [Cup o' Go](https://cupogo.dev/) podcast, [episode 123](https://cupogo.dev/episodes/the-greatest-episode-of-all-time).”*

[Full Code](https://github.com/mmudama/misc/blob/master/playground/fun-with-slices/slices.go)
