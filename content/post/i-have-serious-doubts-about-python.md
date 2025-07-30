+++
title = "I have serious doubts about Python"
description = "A non Python dev and his struggles writing reptilian."
author = 'viticlin'
publishDate = 2025-07-30T00:00:00
draft = false
tags = ["python", "modules", "imports"]
summary = "My struggles trying to build a somewhat serious project with Python."
+++

My most recent side project is an Arduino Programming Language AST interpreter fully written in Python. 

> The only way I could get around finishing my Bachelor's Thesis was framing it as a side project.

During this time using Python for a real world-ish medium sized project I have gone through multiple phases. All of them
converge to being skeptical about modern computers' _lingua franca_. Or it could just probably be a skill issue on my side.

## My background

Around a year and a half ago my functional bro [^1] arc began, as I changed jobs and Elixir became the most used tool under
my programmer belt. I fell in love with pattern matching, pipes and everything-is-an-expression. Currently I am fully
sold on functional programming and I see it as a superior way of punching modern day cards. 

I think of myself as a computer guy. I want to be able to build systems by telling computers what to do. I should be
able to tickle transistors with a broad set of symbols. So around the time I was wondering if I should try to touch
grass outside of the superior programming paradigm, I came across the oportunity of this project.

## Reality check 

Ok! So let's build a scanner [^2]! I see that Python has a `match` statement and I also learn to not look for stuff
outside of the official Python docs. Seriously, the top two pages for almost any search related to Python in DuckDuckGo
and Google are full of SEO driven pages. Not cool.  

Going back to the `match` statement. It's a statement 😓. I have to rely on mutability if I want to do something with
the result of my operations inside the plethora of `case`s. Oh wait, now I'm scared about the typing being dynamic. This
did not happen with Elixir.

## Type check

I guess I'll have to learn type hints. I've seen Python code without them and I got lost pretty quick, I want to avoid
that. 

Bottom line: it's cool but I feel like it's missing a lot of inference power. The last time I've been doing types
(not typespecs) it was with Rust and it felt effortless. With Python type hinting it feels like I'm typing just to hug
myself and whisper to my ear that everything will be fine. 

## Metaprogramming

Writing an interpreter means you have to conform to a language specification. The [Arduino Language Reference](https://docs.arduino.cc/language-reference/) is lackluster at best. It does not state anywhere that the Arduino Language is a subset of C++. It 
does not explain what is a sketch, or what happens with code that your write outside top level functions. It does not
explain if it supports classes. [^3]  

Anyways, I have to implement many tests, a lot of them look like an input file and an expected output file:

#### Input file

```c
int a = 2;
int b = a + 3;
```

#### Expected output file

In this case, we want to check that the parser does actually parse code correctly.

```python
[
  VariableStmtSpec(
    var_type=TokenSpec(token=TokenType.INT),
    name=TokenSpec(token=TokenType.IDENTIFIER, lexeme='a'),
    initializer=LiteralExprSpec(
      value=TokenSpec(token=TokenType.INT_LITERAL, literal=2)
    )
  ),
  VariableStmtSpec(
    var_type=TokenSpec(token=TokenType.INT),
    name=TokenSpec(token=TokenType.IDENTIFIER, lexeme='b'),
    initializer=BinaryExprSpec(
      lhs=VariableExprSpec(
        vname=TokenSpec(token=TokenType.IDENTIFIER, lexeme='a')
      ),
      op=TokenSpec(token=TokenType.PLUS),
      rhs=LiteralExprSpec(
        value=TokenSpec(token=TokenType.INT_LITERAL, literal=3)
      )
    )
  )
]
```

Well, to run these tests with `unittest` you need to create dynamic tests. I did this creating a `unittest.TestCase`
subclass and reimplement the magic `runTest` method. I still don't fully understand it. I know it would have been much
easier doing some metaprogramming wizardry in Elixir. [^4]

## Closing thoughts

I feel like doing this project is teaching me a lot about programming languages. Not only I'm implementing one, but the
host language is also showing me a different way of doing things. I still feel like Python is overrated for serious
programming™.


[^1]: [Dear Functional Bros by CodeAesthetic](https://www.youtube.com/watch?v=nuML9SmdbJ4)
[^2]: https://en.wikipedia.org/wiki/Lexical_analysis
[^3]: This is just a rant from a language implementer viewpoint. I still think that even just for users, the reference
could be better organized and have way more information.
[^4]: It could really just that I'm a bit more familiar with that.
