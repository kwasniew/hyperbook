# Hyperapp: functional programming in your browser with 500 lines of code
# Chapter 1: Elevator Pitch

## Why do we need another framework?

![Figure: Rube Goldberg Machine - a metaphore for accidental complexity](images/rube-goldberg-machine.jpg)

> We have lost our ability of achieving more with less. We do more with more. Or, in certain cases, we do less with more.

Modern frontend development is too complicated:
* frameworks with 10000k+ LOC, impenetrable to non-core developers
* libraries with huge API surface area
* more and more JS "the bad parts" to learn
* complex tooling to make everything work
* layers upon layers of abstraction on top of the browser
* elaborate performance tricks to improve performance
* code mixing side-effects and application logic 
* testing techniques with mock imports encouraging untestable code

## What is your elevator pitch?

![Figure: Hyperapp Elevator Pitch](images/elevator-pitch.jpg)

```
For JS developers. 
Who need to build highly interactive Web interfaces  
Hyperapp is a functional view and state management framework
That allows for writing easy to understand, performant and testable code
Unlike other JS frameworks
Hyperapp fits in one 500 LOC file with zero dependencies and facilitates programming in a tiny subset of JS "the very best parts"
```
In this book I'll try to back up this bold statement. 

## Learning outcomes

By the end of this book you will learn to:
* develop frontend code using mostly pure functions and object literals 
* setup build tool free development workflow to avoid tooling fatigue
* render views as simple functions, not stateful components
* model application state to make impossible states impossible
* change application state with pure functions
* talk to HTTP APIs and stream server events without callbacks, promises, async/await, observables in the user space
* describe all side effects (e.g. random number generation) as data structures
* optimize load time and runtime performance 
* test your application at the unit and integration level
* deploy your code to production
* render frontend views on the server
* route between different pages
* integrate with 3rd party libraries
* appreciate the benefits of minimalist approach to software development