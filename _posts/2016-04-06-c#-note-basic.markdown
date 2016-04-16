---
layout: post
title:  "C# Note: Basic"
date:   2016-04-06 0:0:0 +0800
categories: C#
---

### C\#

C# is a general-purpose, type-safe, object-oriented(encapsulation, inheritance, and polymorphism) programming language.


### Basic:

Avoiding conflicts: prefix with @

~~~ csharp
class class  {...}      // Illegal
class @class {...}      // Legal
~~~

***

Verbatim string literals: prefix with @

~~~ csharp
string a2 = @ "\\server\fileshare\helloworld.cs";
~~~

***

Interpolated string: A string preceded with the $ character

~~~ csharp
int x = 4;
Console.Write ($"A square has {x} sides"); // Prints: A square has 4 sides
~~~

***

You can’t explicitly delete objects in C#, as you can in C++. An unreferenced object is eventually collected by the garbage collector.

***

To pass by reference, C# provides the ref parameter modifier. In the following example, p and x refer to the same memory locations:

~~~ csharp
class Test
{
  static void Foo (ref int p) {
    p = p + 1; // Increment p by 1 // Write p to screen
    Console.WriteLine (p);
  }

  static void Main()
  {
    int x = 8;
    Foo (ref x); // Ask Foo to deal directly with x
    Console.WriteLine (x); // Ask Foo to deal directly with x
  }
}
~~~

***

The out modifier: An out argument is like a ref argument, except it:

* Need not be assigned before going into the function
* Must be assigned before it comes out of the function

***

The params modifier: The params parameter modifier may be specified on the last parameter of a method so that the method accepts any number of arguments of a particular type. The parameter type must be declared as an array.

***

Optional parameters: A parameter is optional if it specifies a default value in its declaration. Optional parameters cannot be marked with ref or out. Mandatory parameters must occur before optional parameters in both the method declaration and the method call (the exception is with params arguments, which still always come last).

***

Null-Coalescing Operator: The ?? operator is the null-coalescing operator. It says “If the operand is non-null, give it to me; otherwise, give me a default value.”

~~~ csharp
string s1 = null;
string s2 = s1 ?? "nothing"; // s2 evaluates to "nothing"
~~~

***

Null-conditional operator: The ?. operator is the null-conditional or “Elvis” operator, and is new to C# 6. It allows you to call methods and access members just like the standard dot operator, except that if the operand on the left is null, the expression evaluates to null instead of throwing a NullReferenceException.

~~~ csharp
System.Text.StringBuilder sb = null;
string s = sb?.ToString(); // No error; s instead evaluates to null
~~~

The last line is equivalent to:

~~~ csharp
string s = (sb == null ? null : sb.ToString());
~~~

***

foreach loops:

~~~ csharp
foreach (char c in "beer")
  Console.WriteLine (c);
~~~

***

Expression-bodied methods:

~~~ csharp
int Foo (int x) { return x * 2; }
int Foo (int x) => x * 2; //expression-bodied method
~~~
