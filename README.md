# async-in-out
a practical show it, explain it, modify it, guide to asynchronous input/output programming in python

# asyncio

asyncio is designed to allow you to structure your code so that when one piece of linear single-threaded code (called a “coroutine”) is waiting for something to happen another can take over and use the CPU.

It’s not about using multiple cores, it’s about using a single core more efficiently

subroutines have one stack of frames per thread. 

<img src="https://bbc.github.io/cloudfit-public-docs/images/asyncio/Stack3.svg">

in asyncio, instead, each thread has a `event_loop` object. The event loop contains within it a list of objects called Tasks. Each Task maintains a single stack, and its own execution pointer as well.


## References

https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-1
