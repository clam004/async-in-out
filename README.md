# async-in-out
a practical show it, explain it, modify it, guide to asynchronous input/output programming in python

# asyncio

asyncio is designed to allow you to structure your code so that when one piece of linear single-threaded code (called a “coroutine”) is waiting for something to happen another can take over and use the CPU.

It’s not about using multiple cores, it’s about using a single core more efficiently

### subroutines have one stack of frames per thread. 

consider when you have a bug or error in your code.

```python
# bug.py

def a_func(x):
    return x - '2'
    #return x - 2

def main():
    some_value = 12
    some_other_value = a_func(some_value)

main()
```

output

```bash
async-in-out $ python3 bug.py
Traceback (most recent call last):
  File "/Users/async-in-out/bug.py", line 9, in <module>
    main()
  File "/Users/async-in-out/bug.py", line 7, in main
    some_other_value = a_func(some_value)
                       ^^^^^^^^^^^^^^^^^^
  File "/Users/async-in-out/bug.py", line 2, in a_func
    return x - '2'
           ~~^~~~~
TypeError: unsupported operand type(s) for -: 'int' and 'str'
```

the call stack above has the same order as the lower right layers in the image below 

<img src="https://bbc.github.io/cloudfit-public-docs/images/asyncio/Stack3.svg">

When we begin execution of this code the stack is initialised as an empty Last In First Out area of storage in memory, and execution starts at the final line (main()).

1. It adds a new “frame” to the top of the stack. This is a data structure that can be thought of as containing anything that is put on the stack after it.
2. It adds a “return pointer” to the top of the stack. This is an address that tells the interpreter where execution needs to resume from when the function returns.
3. It moves execution so that the first line of the function is the next instruction to be executed.

This pattern is followed by almost all code that it written in traditional programming languages. Multithreaded coding slightly alters this by having a separate stack per thread, but otherwise it’s pretty much exactly the same.

Asyncio, however, works a little differently.

### coroutine calling

in asyncio, instead of one stack of frames per thread, each thread has a `event_loop` object. The event loop contains within it a list of objects called `Tasks``. Each `Task` maintains a single stack, and its own execution pointer as well.

the event loop can only have one Task actually executing (the processor can still only do one thing at a time, after all), whilst the other tasks in the loop are all paused. The currently executing task will continue to execute exactly as if it were executing a function in a normal (synchronous) Python program, right up until it gets to a point where it would have to wait for something to happen before it can continue.

Then, instead of waiting, the code in the Task *yields control*. This means that it asks the event loop to pause the Task it is running in, and wake it up again at a future point once the thing it needs to wait for has happened. The event loop can then select one of its other sleeping tasks to wake up and become the executing task instead. 

if none of them are able to awaken (because they’re all waiting for things to happen), then it can wait.

This way the CPU’s time can be shared between different tasks, all of which are executing code capable of yielding like this when they would otherwise wait.

<img src="https://bbc.github.io/cloudfit-public-docs/images/asyncio/EventLoop.svg" >

This execution pattern, where code control moves back and forth between different tasks, waking them back up at the point where they left off each time is called “coroutine calling”, and this is what asyncio provides to Python programming, as a means to ensure that CPUs sit idle less of the time.

Anything dealing with http or other internet traffic protocols is almost guaranteed to be IO bound. coroutine calling works well for IO-bound code, where long pauses are expected to wait for something else (often another computer) to respond to a request.

An event loop cannot forcibly interrupt a coroutine that is currently executing. A coroutine that is executing will continue executing until it yields control. The event loop serves to select which coroutine to schedule next, and keeps track of which coroutines are blocked and unable to execute until some IO has completed, but it only does these things when no coroutine is currently executing.

## References

https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-1
