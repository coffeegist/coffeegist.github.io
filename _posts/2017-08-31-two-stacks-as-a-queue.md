---
layout: post
title: Two Stacks as a Queue
category: Gist
tags: [code-challenge, python]
---

Before starting work each day, I like to set aside some time to warm my brain up by completing a small coding/algorithmic challenge. These challenges are often very entry-level, but they do a good job of making me really think through what I'm doing. Today's problem asked me to implement a queue using two stacks. Interesting...

## Data Structures

#### Queue

A queue is an abstract data type that maintains the order in which things were added to it, allowing the first elements that were added to be the first elements that get removed (think a drive-thru line). This means that this is a FIFO, or First-In-Last-Out, data structure. Queues ususally have two main operations; Enqueue, and Dequeue. Enqueue will insert an element into the queue, and Dequeue removes an element.


#### Stack

A stack is another abstract data type, but instead of being a FIFO data structure it is a LIFO data structure. You guessed it, LIFO stands for Last-In-First-Out. Picture a stack of plates. The last plate that gets put on the stack is the first plate that is removed. It's the same idea with this structure. The two main methods for a stack are Push and Pop. Push places items onto the stack, while Pop removes them.

## The Problem Statement

I was to implement a queue with 3 methods: put, pop, and peek. Put would add elements, pop would remove elements, and peek would show me the data value of the next element to be removed. As I noted before, this problem was interesting because it is asking me to implement a queue from two stacks. Essentially, creating a FIFO data structure out of two LIFO data structures. It just struck me as odd because I've never thought of doing that before. My instincts when needing a queue have always been to just use a FIFO structure. This will require some thought!

Note: No bounds checking is necessary for this exercise (as per the instructions).

## Iteration #1, Naive

I chose `Python 3` for my language of choice, and quickly started writing code, only have thinking of what I was doing (I clearly hadn't had my coffee yet). Before I knew what was happening, I was already finished with the below solution.

```python
class MyQueue(object):
    def __init__(self):
        self.store = []
        self.length = 0

    def peek(self):
        if self.length > 0:
            return self.store[0]

    def pop(self):
        if self.length > 0:
            self.store.pop(0)     

    def put(self, value):
        self.store.append(value)
        self.length += 1
```

Ahh, it's beautiful and that was easy. Wait, I completely ignored the whole problem statement! I used no stacks, only a python list to implement this! At least I remembered what a queue was on auto-pilot. Let's try that again, but this time with feeling.

## Iteration #2, Refined

After making a nice Aeropressed cup of [Counter Culture Coffee](https://counterculturecoffee.com/), I sat back down at my desk and devised a way of creating a queue from two stacks. It felt so backwards trying to come up with this. I could add elements to a stack just fine, but when it came to popping or peeking elements, I needed the reverse of the stack. This is where the second stack comes in.

Whenever I want to peek or pop an element I will need to sequentially load the stack of new elements into another stack (reversing the order), and then peek or pop the last element from stack #2.

Disclaimer: I tried to stick with 'stack-like' methods on Python's list object. `append()` adds elements to the top of the stack, and `pop()` pops them off of the stack. I would have used `self.pull[-1]` in the `peek()` method below, but it felt like cheating since you shouldn't be able to access an item by index in a stack. That's why I did the pop()/append() operations.

```python
class MyQueue(object):
    def __init__(self):
        self.push = []
        self.pull = []

    def transfer(self):
        while len(self.push):
            self.pull.append(self.push.pop())

    def peek(self):
        if len(self.pull) == 0:
            self.transfer()
        item = self.pull.pop()
        self.pull.append(item)
        return item

    def pop(self):
        if len(self.pull) == 0:
            self.transfer()
        self.pull.pop()     

    def put(self, value):
        self.push.append(value)
```

## Coffee Break!

And there we have it! A successful queue using two stacks. This was ran against a suite of test cases and passed them all. Did I miss room for optimizing? Have a suggestion on a way to improve `MyQueue`? Please leave a comment below and tell me what you think! It's time for some more coffee... Happy Hacking!
