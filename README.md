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

## Differences between Records and Classes
Here I will underline the differences between Records and Classes.
### ToString
When we console.log a record it will display the values of the properties in curly braces and
the class will just display the namespace followed by the class name.
### Value and Reference types
When we take two objects we created for the same class with the same values, the compiler will recognize them
as different objects. Because it compares their location in the memory and not their values.
It is different for records. Two different records with the same values will be equal. They are two
different objects but they act as value types even though they are not. The compiler compares each property
in the entire object and if they are all equal then the objects itself are equal.
```
Record1 r1a = new("Grommash", "Hellscream");
Record1 r1b = new("Grommash", "Hellscream");
```
When comparing r1a and r1b objects r1a == r1b => true (also works for != and Equal()) 
ReferenceEqual() will be false.
## HashCode
r1a and r1b will both have the same hashcode. If two objects from the same class have the same values
they will have different hashcode because it is based on the memory location of those objects.
Two records on the other hand will have the same hashcode value because of the values which are the same.

:writing_hand: *This is very useful because if we want to check if a list has any duplicates in it we can
do a LINQ query and say give me all objects with unique hashcode values.*

## Deconstructor
```
var(fn, ln) = r1a;
Console.WriteLine($"{fn}, {ln}"); // Grommash, Hellscream
```
By assigning the record to an anonymous tuple it will deconstruct the properties of the record
into the values of the tuple

## Object copying
```
Record1 r1d = r1a with
{
    FirstName = "Garrosh",
};
Console.WriteLine($"Garrosh's record: { r1d }"); // Garrosh Hellscream
```

We can create copies of records and say take everything from this first object but everything after
the with keyword in curly braces will be changed to new values.

:man_technologist: *The **with** expression is introducted in C#9 and is used to create a copy of a 
record with specified properties*

## Difference between Struct and Record
**Records are Classes**, they just have extra stuff. A struct is a value type, it is not a class which means
it cannot be instantiated, inherited and so on.. Because records are classes we can inherit, instantiate them etc.

## Important Record features
- Records can have methods
- Record properties can be created in various ways
- We can use the class syntax for creating a record
- A record can inherit another record
- A record **CANNOT** inherit another class

### Explicit declaration of values
```
public record Record2(string FirstName, string LastName)
{
    internal string FirstName { get; init; } = FirstName;
    public string FullName { get => $"{ FirstName } { LastName }"; }
}
```
### Methods
```
public record Record2(string FirstName, string LastName)
{
    public string SayHello()
    {
        return $"Hello { FirstName }";
    }
}
```
### Full Properties
private string _firstName;

public string FirstName
{
    get { return _firstName.Substring(0, 1); }
    init { }
}

Records can also have full properties defined.  
:keyboard: *To create a full property faster we can type in the phrase propfull*

### Inheritance
A record can inherit only from another record, not from another class.
```
public record User1(int id, string FirstName, string LastName) : Record1(FirstName, LastName);
```

### Why use Records?
- Benefits  
  Records are extremely easy to setup and require less code to work. One line can replace tens of lines.
- Thread safe  
  If we have two threads working on the same object in the same time, if they both change the value of that object
  ,it can create a conflict and will blow up. Since we cannot change the value of a record then there are no
  problems. 

### When to use Records?
- If we are capturing external data that doesn't change.
- When we have our own API calls. If we are just getting the data we should use records, not classes  
- If we have 10,000 people in the database and return them we can probably populate records with that data
- We can classes that have Record types as properties as well which can oftentimes be useful.
- In general we use Records whenever we are just taking data and not modifying it (**Always when using READONLY data**)

### When not to use Records?
- When using Entity Framework or data that needs to change in any way

### NEVER DO THIS
```
public record Person // No constructor so no deconstructor (BAD!)
{
    public string FirstName { get; **set;** } // set makes this record mutable (BAD!)
}
```
Don't force records to be mutable when they are intended to be immutable.  
:man_technologist: *We should also never create clones just to update the state of objects.*

###### Legend
:man_technologist: - Coding tips I find to be important  
:writing_hand: - Coding conventions  
:keyboard: - Coding shortcuts and bindings  
