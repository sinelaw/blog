---
title: "Generate Javascript classes for your .NET types"
date: "2014-04-08"
tags: 
  - "net"
  - "c"
  - "javascript"
---

We open-sourced another library: ClosureExterns.NET (on [github](https://github.com/bdb-opensource/closure-externs-dotnet) and [nuget](http://www.nuget.org/packages/ClosureExterns.NET/)). It generates Javascript classes from .NET-based backend types, to preserve type "safety" (as safe as Javascript gets) across front- and backend. As a bonus you get [Google closure annotations](https://developers.google.com/closure/compiler/docs/js-for-compiler#types). The type annotations are understood by WebStorm (and other editors) and improve your development experience. Also, if you use Google Closure to compile or verify your code, it will take these types into account. We use it extensively with C#. We haven't tried it with F#, but it's supposed to work with any .NET type.

ClosureExterns.NET makes it easier to keep your frontend models in sync with your backend. The output is customizable - you can change several aspects of the generated code. For example you can change the constructor function definition, to support inheritance from some other Javascript function. For more details see [ClosureExternOptions](https://github.com/bdb-opensource/closure-externs-dotnet/blob/master/ClosureExterns/ClosureExternsOptions.cs).

## [](https://github.com/bdb-opensource/closure-externs-dotnet#getting-started)Getting Started

First, install it. Using **nuget**, install the package [ClosureExterns](http://www.nuget.org/packages/ClosureExterns.NET/).

Then, expose a method that generates your externs. For example, a console application:

```
public static class Program
{
    public static void Main()
    {
        var types = ClosureExternsGenerator.GetTypesInNamespace(typeof(MyNamespace.MyType));
        var output = ClosureExternsGenerator.Generate(types);
        Console.Write(output);
    }
}
```

You can also customize the generation using a `ClosureExternsOptions` object.

## [](https://github.com/bdb-opensource/closure-externs-dotnet#example-inputoutput)Example input/output

### [](https://github.com/bdb-opensource/closure-externs-dotnet#input)Input

```
class B
{
    public int[] IntArray { get; set; }
}
```

### [](https://github.com/bdb-opensource/closure-externs-dotnet#output)Output

```
var Types = {};

// ClosureExterns.Tests.ClosureExternsGeneratorTest+B
/** @constructor
*/
Types.B = function() {};
/** @type {Array.<number>} */
Types.B.prototype.intArray = null;
```

For a full example see [the tests](https://github.com/bdb-opensource/closure-externs-dotnet/blob/master/ClosureExterns.Tests/ClosureExternsGeneratorTest.cs).
