# Chapter 3 - Refactoring

We now move onto the next chapter.

**Warning; Here be dragons.**

This is the chapter where objective BYOND-isms go out the window, and everything from here on out is going to be some pretty opinionated things.
If you're just making a simple game, you likely have no need of any of this. If you follow the design of codebases like Space Station 13, you will use far more time on your game than you otherwise would, for the potential payout of a structured, modular, and robust codebase.

We're going to talk about crazy things like:

- Putting variables on the lowest level that needs it.
- Modular code design and avoiding code reuse
- Global management subsystems
- **The SpacemanDMM linter & Auxtools debugger**
