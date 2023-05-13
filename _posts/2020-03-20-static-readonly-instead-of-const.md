---
layout: post
title: Static Readonly instead of Const in C#
date: 2020-03-20
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Let's take a look at how constants work in C# and why using static readonly might be better some cases.
tags:
- dotnet
- C#
---

I have always considered it a best practice to use static readonly instead of const in C#. Here’s why…

The `const` keyword tells the compiler to replace the constant token in your code with the literal value you have defined for the constant. You can see this if you look at the IL:

The C# code…

``` csharp
const string HELLO = "Hello";
...
Console.WriteLine(HELLO);
Console.WriteLine(HELLO);
```

Compiles to the IL…

``` 
.field private static literal string HELLO = "Hello"
...
IL_0001:  ldstr      "Hello"
IL_0006:  call       void [System.Console]System.Console::WriteLine(string)
IL_000b:  nop
IL_000c:  ldstr      "Hello"
IL_0011:  call       void [System.Console]System.Console::WriteLine(string)
```

Notice that the literal "Hello" is loaded twice.

Using static readonly provides us similar semantics–a global value that cannot be changed–but rather than references being replaced with literals by the compiler, it remains a reference.

This C# code…

``` csharp
static readonly string WORLD = "World";
...
Console.WriteLine(WORLD);
Console.WriteLine(WORLD);
```

Compiles to the IL…

``` assembly
.field private static initonly string WORLD
.method private hidebysig specialname rtspecialname static void .cctor() cil managed
{
  .maxstack  8
  IL_0000:  ldstr      "World!"
  IL_0005:  stsfld     string StringInterning.Program::WORLD
  IL_000a:  ret
}
...
IL_0017:  ldsfld     string StringInterning.Program::WORLD
IL_001c:  call       void [System.Console]System.Console::WriteLine(string)
IL_0021:  nop
IL_0022:  ldsfld     string StringInterning.Program::WORLD
IL_0027:  call       void [System.Console]System.Console::WriteLine(string)
```

Notice that the literal string "World" is only loaded into memory once, and the reference is used in the calls to `Console.Writeline()`.

Why does this matter?

It makes a big difference with Strings. Strings eat up memory (specically the large object heap) and string comparison is slower than a reference comparison. This is also the reason it is best practice to use `string.Empty` instead of `""`.
Interning is supposed to alieviate these performance issues, but isn’t perfect. For example…

``` csharp
const string HELLO = "Hello";
string s1 = new StringBuilder(HELLO).Append(" World");
```

…will not use the interned copy of "Hello" in the constructor of `StringBuilder()`, nor will it intern the literal " World" for later use. This is because the compiler will only automatically intern literal assignments and reference the interned copy for assignments. For example:

``` csharp
string foo1 = "Foo";
string foo2 = "Foo";
```

In the above example, there will only be one copy of "Foo" in memory, and both foo1 and foo2 will point to the same reference in the intern table.

For these reasons, I always (well, almost always) use static readonly string rather than const string.

Unfortunately, we can’t always use static readonly in place of const. For example, default values for optional parameters and values assigned to attribute properties must be const/literals.

For consistency, I take the same approach with numeric and enum constants as well. Though the performance gains are not as significant as with strings, it keeps the coding style consistent and therefor easier to read.

Cheers!
