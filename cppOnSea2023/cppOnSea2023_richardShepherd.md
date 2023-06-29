# CppOnSea 2023 trip notes (Richard Shepherd) 

## Tuesday, 27th June 

### Workshop – Introduction to C++ Coroutines 

Coroutine language support is available in c++20. 

Phil Nash presented this course at short notice after the original speaker went covid. This turned out to be great value for money as the slides used were for a two-day course intensively crammed into one! 

There are several libraries that provide nice helper functions, classes, constraints and concepts, but no std libraries yet. They will be in a version (hopefully c++26) after the language support has been released and absorbed by the community. 

The workshop looked in detail at `co_return`, `co_await` and `co_yield`. 

Several approaches were used to explain ways to think about them, including presenting pseudo-code of what these keywords might be considered syntactic sugar for i.e. the one-line with the new key word replaced 10 lines of boilerplate code calling callbacks. 

This also led nicely into understanding what callbacks those keywords expected to use. These methods are the parts that developers create to specify the policies for what the co_ keywords will do. They are added as part of the interface of the return type of the coroutine (analogous to, and sometimes actually, a std::future) or the type of the expression following the keyword. 

Using godbolt’s compiler explorer for the exercises was very effective (continuously auto-compiling and running the code I attempted to write) – as well as a reminder that the mp-coro library was available as part of godbolt (and was written by the author of the slides) 

Unfortunately, the discussion around coroutines interacting with threads to get async, parallel behavor was not an area Phil had a lot of experience and the result was a bit hand-wavey, but it seemed like you should be able to do the things you’d expect: having a thread prepare the next results while you’d yielded the previous value to the caller. 

The `co_yield` discussion led onto looking at a popular use: generators, which reduces the boilerplate required to construct non-collection iterators and ranges. 

If time had allowed, I would have liked to have seen more examples of the use of co_await and co_return to understand the motivations and possible designs that these mechanisms opened up. 

## Wednesday, 28th June 

 

## Thursday, 29th June 

 

## Friday, 30th June 

 

 