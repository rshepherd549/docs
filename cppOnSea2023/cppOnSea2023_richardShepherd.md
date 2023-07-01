# CppOnSea 2023 trip notes (Richard Shepherd) 

## TLDR - Takeaways from the week



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

### Keynote: All the safeties (Sean Parent)

Sean highlighted the recent increased legislation, particularly in the US, but also in the EU, increasing the safety liability on developers and companies for their products. Despite the fact that the legislation made almost no effort to define what memory safety was.
Older, academic work (using the Frame rule) did define the problems more comprehensively, clarifying the misuse of memory e.g. just because it has been initialized and is within your process doesn't mean that access to it via, for instance, an adjacent variable, isn't wrong.
No language or system could be completely safe, if it was powerful enough: it is always possible to recreate unsafe violations within a safe environment, e.g. recreate memory violations in your process' array workspace.
Sean's work at Adobe includes the Val language, which sounds like it is focusing on a safer subset of c++.
He was also keen on the ideal of moving software engineering towards:
- library of proven components
- declarative forms for combining components.

### So you want to use C++ Modules - cross platform (Daniela Engert)
A 10 minute recap of the Module mechanism: the implementation and interface files and the module, import and export keywords.
Most of the talk discussed the issues hit on gcc, msvc and/or clang when trying to port typical existing libraries to modules. gcc support was the weakest, almost unusable because it didn't support the header unit interfaces (which sounded like the fallback mechanism of modules for anything that you used to do with #include that wasn't supported by nicer parts of Modules).
MSVC was the most complete, but also perhaps a little over-tolerant. Clang was very strict.
A lot of painstaking work (e.g. a month) was required to get a successful build. This seemed to be a mixture of patching and tweaking the toolchain for doing the builds and of tweaking the source code to be more amenable to the Modules mechanisms.
When successful, the improvement in build times (e.g. for pulling in large chunks of stl or sdl) was impressive - 2 or 3 orders of magnitude faster.

### SYCL (Joel Falcou)
SYCL is an emerging standard language for abstracting access to multiprocessor environments (cpu, gpu, fpga).
Intel has a fully feature implementation available as part of its OneAPI project, but the ideas should apply to other vendors and environments.
The sycl dsl appeared to be mostly about requesting working space (e.g. buffers) in a virtual environment which could then be used with normal c++ code and algorithms (including parallel algorithms). The wrappers of the buffers would be allow the environments to do their specific setup or redirection.
The sound quality wasn't good enough for me to catch more of th details.

### C++ Features you might not know (Jonathan Muller - ThinkCell)
An entertaining talk pointing out dark corners of c++ e.g. 17[array] == array[17].
- unary `+` on a value will promote its type. Possibly unexpected, sometimes useful. Unary '-' does the same
- `!=` is autogenerate from `==` in c++20 (related to `<=>`)
- switch can have never used code before its first case. Could be usdefully used for `using enum`
- `valarray` has many useful features but is almost entirely unknown and ignored

### Templates made easy with C++20 (Roth Michaels)
A good explanation of concepts and, in particular, `requires`, with good examples.
A question I've been pondering for a while: how to add static tests into the code to regression test constraints i.e. static-assert that some piece of code is not compilable, is possible with nested `requires` where it would have been pretty messy with SFINAE.

### A tour of polymorphism techniques (Andrew Marshall)
A survey of several techniques and their performance implications, let down by awkward live coding that made it difficult to follow the thrust of the talk.
- original inheritance - relatively expensive virtual function calls
- crtp
- variant (gcc11 poor implementation, gcc12 good)
- for platform specific variant use conditional inclusion

### Lightning talks

## Thursday, 29th June 

### Coroutine intuition - Roi Barkan

Recap that the ideas were described in the 50s and 60s and then fell out of fashion until the 2000s.
*My thought: did they fade because structured programming ideas emphasized logic that was easier to reason about, and coroutines were more complex - with the danger of rebuilding spagetti goto code!?*

Compared to the workshop, this talk was more concerned with discussing when application developers would use co_await etc.
What sort of designs would benefit, or not, from coroutines?
Possibilities include:
- complex component interactions
- applications with massive concurrency
- separating algorithm logic from execution policy

In some ways, it is the opposite of lambdas which allow you to bring together the algorithm and the functionality it acts on.

co_yield has fairly straight forward mental model - for generators that preserve their state between calls.

I still struggle to see the whole picture for co_await. My takeaway so far is to consider it a tool for cooperative multitasking, but tis seems more general than that.

With the possibility of passing control from one coroutine handler to another, you get something like a linked list of coroutine frames, forming a stack (compared to the normal execution stack).

It was frequently mentioned that suspending/resuming coroutines was very efficient, potentially the lightest weight way to jump between control flows. It was described as being about the same cost as a virtual function call *(ironic as the talk last night spent most its time looking for ways to avoid the cost of virtual function calls)*.

### New algorithms in c++23 - Conor Hoekstra

My favorite talk.
A nice review of iterator based algorithms available since c++98, range algorithms since c++20, and the adapters and views added in c++23.
Between then, they allow a lot more *collection-oriented* programming, with operations carried out one after the other on oen or more collections or views. All of it was reaching towards more composable operations that are seamlessly available in other languages (apl etc at the most extreme, but haskell as a nice, readable target).

c++ use of piping for composition is limited. Operations that take a variadic number of arguments don't support it, even for the two parameter case e.g. zip.
Circle does provide an enhanced pipe operator (`|>`) which allows operations to use `$` one or more times as a placeholder and thus enable piping of far more of the operations.

Several examples of problems, their solution in imperative c++ and their transformation to collection-oriented were given, including the sushi-for-two competitive coding challenge.
Actual development of a solution was often via a more succinct language or pseudo code, before translating to the functions available in c++.

There were lengthy discussions after the main talk, concerning the tradeoff between time to understand and think about a problem and its solution to use a collection based solution vs leaping in to an imperative solution which superficially matches the problem description or the first thoughts about a manual solution.
This included recognition that edge case bugs were far more likely in the imperative code, and it was no easier to reverse engineer intent out of nested loops and branches for non-trivial code.
The collection based approach made the steps more explicit, allowed easier reasoning because you could think about the whole collection at each stage, but without the performance dangers of materializing whole collections repeatedly - because of ranges lazy evaluation.

### Sponsor - Tipi - combining git and make
The aim was to reduce build times for developers where many of them would be rebuilding the same things.
Seemed to be an experiment to store intermediary binaries, relying on hashing rather than build datetime.
This was combined with compression which averaged storing 10gb binaries in 100mb, so many could be stored in lfs.
When it worked, those parts of the build had their usual build time replaced by the file transfer and decompression time, which was frequently quicker.

### The C++ RValue lifetime disaster
This was a good discussion of some of the history of const-ref extending the lifetimes of rvalues and how this has played out with newer explicit rvalue types. Basically badly, with little that can now be done to improve the situation because of backward compatibility.

roughly the following promotions happen implicitly:

short lifetime      long lifetime

const && -------->  const &
^        ----/      ^
|  ----/            |
&&                  &

which leads to easily dangling pointers that aren't flagged by the compiler.
It would be much nicer if, and break just about no real cases if:

short lifetime      long lifetime

const && <-------   const &
^           \--      ^
|              \--- |
&&                  &

The speakers company used several types and macros to wrap up and avoid some of the problems of returning const-ref multiple times losing the lifetime extension, or to forward rvalues better. Ithink...

### Why clean code is not the norm - Sandor Dargo
A brief discussion of the difficulty of defining quality or clean code, ,with reference to Zen & the art of Motorcycle Maintenance.
The most useful definition of quality came from the CISQ: security, reliability, maintainability, performance.
Clean was described as:
- easy to understand (code flows, roles, resposibilities, purpose)
- easy to change (concise public interfaces, testable and tested, predictable, single responsibility)

Except that it my appear more costly in the short term (attention, effort, care, time, learning) *(I think skills and ability should be added)*
Users want solutions to their immediate problems, hence managers too. But both disappointed that future development takes longer or not possible.

Some studies measured the costs and benefits, which suggest that doing the wrong thing well is a better risk than doing the right thing badly.
PIC

With the number of junior developers increasing significantly, we need to expect to mentor and demonstrate discipline and good examples.
The speaker suggested:
- we should go above and beyond e.g. your PR could clean up nearby code, not just add clean code
- be willing to learn, communicate, explain and educate
- be willing to say no, and even to leave
- vaguely answered a question that disagreeing devs should talk until they didn't disgree.

*(There seemed some utopian privilege in assuming that we were correct and not having to learn or trust their leads)*

### Why loops end - Lisa Lippincott
This was a much more academic example of proving that a program had a specific property.
The main example was proving that the following loop would terminate (before the inevitable heat-death of the universe!) if `a<n`
```cpp
int i = a;
while (i < n)
  ++i;
```
This used pseudo c++ to discribe theorem interfaces, implementations and claims, to describe pre-conditions and (eventually) establish post-conditions.
The discussion methodically looked at simpler cases (e.g. `while(false){}`) and low, known values of `a` and `n` and built up the logical machinery to cover all singe digit values, then double digit values.
To viewers without experience (just about everyone) it seemed highly repetitive and may have benefited from some scene setting of what was intended or an overview of the process and what it achieved.
The questions suggested folks had got the jist and were interested in how it might be automated to be more widely used, or how it might compose to scale to more complex code and requirements.

ref: prev talk: The foundations of arithmetic (cppCon2022)

### Lightning talks
Interesting 5 min example from Nanopore using a recursive template inheritance to have a Point class and Point class with a data package of info.
Ref: Impossibly something c++ delegates?

Another interesting talk from Dr Allesandria Palizzia introduced recently introduced international standard for Mental Health at Work.
One aspect was management of each risk: Elimination, or Substitution, or Modification, or Administration, or Protection.
This was also the subject of a full talk the following day, on IT related burnout.

## Friday, 30th June 

### AI assisted Software Engineering
The talk shared the experience of guiding chatGPT (via OpenAI interface) assist a developer.
This was made more specific as assisting in analysis, rather than creative work.
The speaker has been involved in developing CWhy at NVidia.
This focused on tooling to iterate on:
- examining a situation
- looking for references to further content
- retrieving and including that content in the context for the new round of examination.

The problem of unit testing AI assisted processes was discussed, with the immediate current difficulty of reproducibility. Even with a very low randomness temperature, it was very difficult to track or fix the version of the llm being used.
The llm are all limited by having a limited size context, so they eventually start to forget some of the earlier discussion.

The specific task was to create a parallelizible `unique` algorithm. This a semi-serious experiment, not a discussion of the llm technology.
The initial responses were restatements of the problem, or snippets of code - often quoted the nvidia thrust parallel unique algorithm, which it had to be repeatedly told to not use.

Several goes at quite lengthy conversations were required to arrive at the desired single-pass, no-allocation implementation, which clearly relied heavily on the guide knowing what was possible and felt correct.

Conclusions:
- alternate between two modes of prompting:
  1. brainstorming, requested several options, without code
  2. refining an option with code
- only have one objective at a time and only change vary or change one thing at a time
- OpenAI has functions for connecting to models

### Polymorphism
The polymorphism referred to good interfaces between a variety of components (e.g. services), so that additional components could be inserted into a pipeline without changes to other components.
The particular library allowed a collection of pipeline pieces to be assembled (via the pipe operator) at compile time.
If applied to data known at compile time - or at least the tagged properties of the data known - then the pipeline stages would only be instantiated if needed at compile time.

### Simulating trading system and exchange - sponsor IMC
Brief explanation of:
- markets and trading
- the sort of structures which describe orders and responses
- the realities of lags at different stages that are modelled in the simulation
- where the trading strategies fit in (but proprietary, so you'll have to apply to join to find out more)
- simulation concentrated on maximizing speed of trades, being first; did not simulate adversarial traders
- did operate in a continuously changing environment of regulations which attempted to maintain fair access to the markets.

### Speeding Date: Fast Calendar Calculations
An amusing and interesting presentation of the speaker's hobby research.
This has including significant improvements to the leap year and date calculations of several libraries, including std::chrono.
The history discribed the development of the western calendars and their calculations.
The most interesting area which I hadn't come across before was the Christian Zeller computational calendar (1832?) which was easier to compute with and easy to translate to/from gregorian calendar.
The key improvements that the speaker et al had been involved with was clarifying the Euclidean affine calculations used, and applyng them more completely.
This allowed:
- divisions and mod calcs to be sped up to multiplications and division by power of two
- branches to be moved (for instance, teh Zeller calendar moves Jan and Feb to the end of the year, with the result that the varying length of Feb doesn't need logic to handle it)
- overall, almost 10* faster than some common implementations and 2* faster than the previous best.

### Building Interfaces that are hard to use incorrectly
- relaxing the precondition of functions often doesn't help, as it complicates the post conditions and results (statuses, exceptions etc that need dealing with)
- tightening pre-conditions via types is preferable
- ref: strong types (Bjorn Fahller library)
- gathering connected parameters into structures - either need constructed, or could use designated field names in c++20
- use variant with visitor to check that all enum values are managed
- use passkey token rather than friendship to provide more controlled, limited access
- different types to represent different states which have only appropriate functionality available
- even with move, it tends to leave dead variables littering the code
- use-after-move analysis not supported by the language, but covered by some checkers
  - not as good as rust
- more stable interfaces describe what instead of how

### Lessons from High Performance Computing
The speaker described his work with highend audio components with demanding processing time constraints.
He gave an overview of performance measuring, tracking, profiling and analysis.
- cpu timestamp has a much lower overhead and more comparable across platforms than std::chrono stopwatch
- cache misses were of particular importance: l1, l2, let alone main memory
- SIMD can be esy to demonstrate a 4* improvement
  - can often be enabled just with compiler settings, but may need -ffast-math enabled (which will not give ieee precision)
- important to regression test so that changes with new libraries or compiler versions are spotted asap
  - this can mean rereviewing and reversing previous optimizations if they targeted particular side effects rather
    - preferable to improve the algorithm, remove unnecessary work, set things up better so that the compiler will always have a better opportunity
- low level control of calculations in tight loops can be effective e.g. post ordered DFS, and minimal use of memory registers
  - but some examples of clever allocation resulted in slower performance on some platforms if the logic took longer than the gain.

