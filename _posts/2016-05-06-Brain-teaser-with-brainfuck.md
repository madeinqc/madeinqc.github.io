---
layout: post
title: Brain teaser with Brainfuck
comments: true
description: Do you know brainfuck? Is your logic good? It's time to find out! I propose you a small challenge in this esoteric language.
image: /public/img/2016/05/06/Brain-teaser-with-Brainfuck/brainfuck.png
---

Do you know brainfuck? Is your logic good? It's time to find out! I propose you a small challenge in this esoteric language. Why brainfuck? Because it's easy to learn and it will test your logical thinking.

If you know brainfuck, you can <a href="{{ site.baseurl }}{{ page.url }}#challenge">go directly to the challenge</a>. Else, bear with me, it's simple... or is it?

![Brainfuck code](/public/img/2016/05/06/Brain-teaser-with-Brainfuck/brainfuck.png)

<!--more-->

Brainfuck is an esoteric language meant to be minimalist with a very small compiler (the smallest is below 100 bytes). It's so minimalist that it only contains 8 commands.

The language manipulate the data in an array of at least 30,000 byte cells initialized to zero. When you reach the limit of the array, the result is undefined, but most of the implementation loop around. You can manipulate the array with the 8 following instructions:

```
> — Increment the data pointer (point to the next cell in the array)
< — Decrement the data pointer (point to the previous cell in the array)
+ — Increase the value at the data pointer
- — Decrease the value at the data pointer
. — Output the byte at the data pointer (as an ASCII character)
, — Read 1 byte of input (read the ASCII value of 1 character)
[ — If the byte at the data pointer is zero, jump to the matching ']' command, else continue normally.
] — If the byte at the data pointer is non-zero, jump back to the corresponding '[' command, else continue normally.
```

As you might have guessed, `[` and `]` are used as loops, always with the same condition:

``` js
while (currentValue != 0) {
  // code
}
```

We can use them to multiplicate. Note that all the characters that are not commands are considered as comments.

```
+++++   5 in cell 1
[       start loop
  -     minus one on cell 1
  >     go to cell 2
  +++   add 3 to cell 2
  <     go back to cell 1 for comparison
]       The loop will be executed 5 times so cell 2 will contain 3 * 5
```

Here is the famous **Hello World!** implementation.

```
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```

Refer to the [Wikipedia article](https://en.wikipedia.org/wiki/Brainfuck) for more information.

<strong id="challenge">
  Here come the challenge!
</strong>

I suggest you to use this [JavaScript Brainfuck Interpreter](http://www.iamcal.com/misc/bf_debug/) as it also come with a debugger.

You need to make a program that will take a single char as input and if it is between `0` and `3` inclusively, print the number in lowercase letter.
If you receive `0`, print `zero`, if `1`, print `one`, etc. All other entries should be ignored and print nothing.

Leave your answer in comments (commented, compact or both). Obviously, don't read comments if you haven't done it yet.

Game on!
