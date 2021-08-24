# ASP.NET Core async web api
###### Dorijan Fabijanic
These notes encapsulate why you should build your API to be asynchronous, how to do it and what are the main benefits.
This is mostly compiled of tutorials all over the internet but primarily Kevin Dockx tutorial on Pluralsight called:
[Building Async API with ASP.NET Core](https://app.pluralsight.com/library/courses/building-async-api-aspdotnet-core/table-of-contents).

## Overview
Making your web API to be asynchronous does not improve performance but it improves scalability. Vertical scalability to be precise.
When you have a web application and add more servers to handle traffic capacity, that is called **Horizontal scaling**.
When you write an API in a way that increases resource utilization that is called **Vertical scaling**.
Writing an API in an asynchronous way improves vertical scalability of the API. It makes it so that the API
can handle more traffic without implementing additional server capacity. 

### Handling *Synchronous* vs *Asynchronous* requests
When someone makes an synchronous request towards our API, ASP.NET Core requires a thread from the thread pool.
Lets imagine it is a call towards the database (I/O call). This thread will be blocked until the I/O operation is finished.
When a second or a third request comes in, it requires a new thread from the thread pool. Now lets say we have two threads total
in the thread pool. The third request that comes in has to wait until one of the two I/O calls finishes in order to start the operation.
That will make the system slow and will be bad for the user. If it takes too long for the first request to finish, the user will even be prompted
with an error, usually 503 service unavailable.

When someone makes an asynchronopus request towards our API, ASP.NET Core still requires a thread from the thread pool. Instead of 
blocking the thread while the operation is executing, it simply returns the thread to the thread pool. It is only when an I/O call is finished
that the thread becomes unavailable again. But in the meantime, the thread can be used for other I/O requests. So if a new request comes in it can 
be handled without any issues. It improves scalability because it allows the API to use its resources more efficiently than when it uses synchronous handling.

### I/O Bound work vs Computational bound work
I/O Bound work is something that regards more when building async APIs. If you are asking the question:
*"Will my code be waiting for a task to be complete before continuing?"*, you want I/O bound work. It is used
when trying to write something to the file system, database or perform network calls hence why it is used
when building async APIs using ASP.NET Core.

Computational bound work is used when you are asking the question:
*"Will my code be performing an expensive computation*, then you are using computational bound work.
You want your code to perform other tasks while the expensive computation is being executed. This type of work
is typically most used in client side async applications to allow other parts of the system to work while you perform
some computation. Lets say you have client side sorting. While you sort a table, you want the rest of the application 
to be available for use for the user. That is Computational bound work when discussing async operations.

!!!!**DON'T USE ASYNC ON THE SERVER SIDE FOR COMPUTATIONAL BOUND WORK**!!!!
This causes the decline of scalability and performance.

## Async / Await
Making a method with the async modifier ensures that the await keyword can be used inside the method once or many times.
We are also transforming the method into a state machine.

The await operator tells the compiler that the async method can't continue until the awaited asynchronous process is complete.
In the meantime control returns to the caller of the async method.

:man_techologist: *When an async method doesn't contain an await operator, the method simply executes as a synchronous method does.*
:man_techologist: *A method that is not marked with the async modifier cannot be awaited.*

## Async Return Types
Lets say we are calling a repository method that performs some operations and queries data from the database. If we call the method with the await keyboard and we make the method to be async, while the operation is performing, the control will return to the caller method which will continue its execution without waiting for the original control to finish. How do we know when the operation finishes then?
That's where return types come in. An async method can have these return types:
*void
    really only advised for event handlers (a button click event in the client)
    bad
    difficult to test
    hard to handle exceptions
    can't notify the caller that it is finished
*Task & Task<T>
    Return Task if no return type otherwise check Task<T>, usualy executes asynchronously
    Task Represents a single operation that returns nothing.
    Task<T> Represents a single operation that returns a value of type T (Task<T>), usualy executes asynchronously
    :man-technologist: *Task represents the execution of the async method, not the result of it!*
    Both Task and Task<T> have a property Status, IsCanceled, IsCompleted and IsFaulted which determine
    the state of the task.
    The state machine manaages the task and the async keyword transforms the method into a state machine!
*Types with accessible GetAwaiter() method (C#7)
    Return any type that has an accessible GetAwaiter method. These are created because Task and Task<T>
    are reference types and they can induce memory allocation in performance-critical paths, and that
    can adversely affect performance
    Supporting generalized return types allows return a lightweight value type.

### Terms and definitions
  * *Thread* - A basic unit of CPU utilization
  * *Multithreading* - One single CPU or single core in a multi-core CPU can execute multiple threads concurrently
  * *Concurrency* - Condition that exists when at least two threads are making progress (Doesn't mean they are being run simultaneously, at the same time)
  * *Parallelism* - When at least two threads are being run simultaneously, we can achieve this when the server has a multi core processor, we can not achieve it on a single core processor

###### Legend
:man_technologist: - Coding tips I find to be important  
:writing_hand: - Coding conventions  
:keyboard: - Coding shortcuts and bindings  
