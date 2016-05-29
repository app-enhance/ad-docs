# Contributing

## Build system

Use a new system called [KoreBuild](https://github.com/aspnet/KoreBuild), which is built using the sake build tools.  
The sake project is available here: https://github.com/sakeproject/sake

## Coding style guidelines

1. Use [Allman style](http://en.wikipedia.org/wiki/Indent_style#Allman_style) braces, where each brace begins on a new line.
   A single line statement block cannot go without braces because of nesting and readability.
2. Use four spaces of indentation (no tabs).
3. Use `_camelCase` for private fields and use `readonly` where possible.
4. Avoid `this.` unless absolutely necessary
5. Always specify member visibility, even if it's the default (i.e. `private string _foo;` not `string _foo;`)
6. Namespace imports should be specified at the top of the file, *inside* of `namespace` declarations and should be sorted
   alphabetically, with `System.` namespaces at the top and blank lines between different top level groups.
7. Avoid more than one empty line at any time. For example, do not have two blank lines between members of a type.
8. Avoid spurious free spaces. For example avoid `if (someVar == 0)...`, where the dots mark the spurious free spaces.
   Consider enabling "View White Space (Ctrl+E, S)" if using Visual Studio, to aid detection.
9. Use PascalCasing to name all our constant local variables and fields.

### Usage of the var keyword

The `var` keyword is to be used as much as the compiler will allow.

!!! note "They are correct:"

```csharp
var fruit = "Lychee";
var fruits = new List<Fruit>();
var flavor = fruit.GetFlavor();
string fruit = null; // can't use "var" because the type isn't known (though you could do (string)null, don't!)
const string expectedName = "name"; // can't use "var" with const
```
!!! warning "The following are incorrect:"

```csharp
string fruit = "Lychee";
List<Fruit> fruits = new List<Fruit>();
FruitFlavor flavor = fruit.GetFlavor();
```

### Use csharp type keywords in favor of .NET type names

When using a type that has a csharp keyword the keyword is used in favor of the .NET type name.

!!! note "They are correct:"

```csharp
public string TrimString(string s)
{
    return string.IsNullOrEmpty(s)
        ? null
        : s.Trim();
}

var intTypeName = nameof(Int32); // can't use csharp type keywords with nameof
```

!!! warning "The following are incorrect:"

```csharp
public String TrimString(String s)
{
    return String.IsNullOrEmpty(s)
        ? null
        : s.Trim();
}
```

### Use only complete words or common/standard abbreviations in public APIs

Public namespaces, type names, member names, and parameter names must use complete words or common/standard abbreviations.

!!! note "They are correct:"
```csharp
public void AddReference(AssemblyReference reference);
public EcmaScriptObject SomeObject { get; }
```

!!! warning "The following are incorrect:"
```csharp
public void AddRef(AssemblyReference ref);
public EcmaScriptObject SomeObj { get; }
```

### Cross-platform coding

Frameworks should work on CoreCLR, which supports multiple operating systems. Don't assume it only run (and develop) on Windows. Code should be sensitive to the differences between OS's. Here are some specifics to consider.

#### Line breaks
Windows uses `\r\n`, OS X and Linux uses `\n`. When it is important, use `Environment.NewLine` instead of hard-coding the line break.

Note: this may not always be possible or necessary.

Be aware that these line-endings may cause problems in code when using `@""` text blocks with line breaks.

#### Environment Variables

OS's use different variable names to represent similar settings. Code should consider these differences.

For example, when looking for the user's home directory, on Windows the variable is `USERPROFILE` but on most Linux systems it is `HOME`.

```csharp
var homeDir = Environment.GetEnvironmentVariable("USERPROFILE")
                  ?? Environment.GetEnvironmentVariable("HOME");
```

#### File path separators

Windows uses `\` and OS X and Linux use `/` to separate directories. Instead of hard-coding either type of slash, use [`Path.Combine()`](https://msdn.microsoft.com/en-us/library/system.io.path.combine(v=vs.110).aspx) or [`Path.DirectorySeparatorChar`](https://msdn.microsoft.com/en-us/library/system.io.path.directoryseparatorchar(v=vs.110).aspx).

If this is not possible (such as in scripting), use a forward slash. Windows is more forgiving than Linux in this regard.


#### Conditional compilation for Desktop/CoreCLR

Almost all development is done for both CoreCLR and Desktop .NET. Some code will be CoreCLR-specific or Desktop-specific because of API changes or behavior differences. The build system has two conditional compilation statements to assist with this:

Desktop:

```csharp
#ifdef DNX451
```

CoreCLR:

```csharp
#ifdef DNXCORE50
```


### When to use internals vs. public and when to use InternalsVisibleTo

As a modern set of frameworks, usage of internal types and members is allowed, but discouraged.

`InternalsVisibleTo` is used only to allow a unit test to test internal types and members of its runtime assembly. Do not use `InternalsVisibleTo` between two runtime assemblies.

If two runtime assemblies need to share common helpers then use a "shared source" solution with build-time only packages. Check out the https://github.com/aspnet/Mvc/tree/dev/src/Microsoft.AspNet.Mvc.Common project and how it is referenced from the `project.json` files of sibling projects.

If two runtime assemblies need to call each other's APIs, the APIs must be public. If it is need it, it is likely that other customers need it.


### Extension method patterns

The general rule is: if a regular static method would suffice, avoid extension methods.

Extension methods are often useful to create chainable method calls, for example, when constructing complex objects, or creating queries.

Internal extension methods are allowed, but bear in mind the previous guideline: ask yourself if an extension method is truly the most appropriate pattern.

The namespace of the extension method class should generally be the namespace that represents the functionality of the extension method, as opposed to the namespace of the target type. One common exception to this is that the namespace for middleware extension methods is normally always the same is the namespace of `IAppBuilder`.

The class name of an extension method container (also known as a "sponsor type") should generally follow the pattern of `<Feature>Extensions`, `<Target><Feature>Extensions`, or `<Feature><Target>Extensions`. For example:

```csharp
namespace Food
{
    class Fruit { ... }
}

namespace Fruit.Eating
{
    class FruitExtensions { public static void Eat(this Fruit fruit); }
  OR
    class FruitEatingExtensions { public static void Eat(this Fruit fruit); }
  OR
    class EatingFruitExtensions { public static void Eat(this Fruit fruit); }
}
```

When writing extension methods for an interface the sponsor type name must not start with an `I`.


### Assertions

Use `Debug.Assert()` to assert a condition in the code. Do not use Code Contracts (e.g. `Contract.Assert`).

Please note that assertions are only for our own internal debugging purposes. They do not end up in the released code, so to alert a developer of a condition use an exception.


## Unit tests and functional tests

Use [xUnit.net](https://xunit.github.io) for all unit testing.

### Assembly naming

The unit tests for the `Microsoft.Fruit` assembly live in the `Microsoft.Fruit.Tests` assembly.

The functional tests for the `Microsoft.Fruit` assembly live in the `Microsoft.Fruit.FunctionalTests` assembly.

In general there should be exactly one unit test assembly for each product runtime assembly. In general there should be one functional test assembly per repo. Exceptions can be made for both.


### Unit test class naming

Test class names end with `Test` and live in the same namespace as the class being tested. For example, the unit tests for the `Microsoft.Fruit.Banana` class would be in a `Microsoft.Fruit.BananaTest` class in the test assembly.


### Unit test method naming

Unit test method names must be descriptive about *what is being tested*, *under what conditions*, and *what the expectations are*. Pascal casing and underscores can be used to improve readability. The following test names are correct:

!!! note "They are correct:"

```
PublicApiArgumentsShouldHaveNotNullAnnotation
Public_api_arguments_should_have_not_null_annotation
```

!!! warning "The following test names are incorrect:"

```
Test1
Constructor
FormatString
GetData
```

### Unit test structure

The contents of every unit test should be split into three distinct stages, optionally separated by these comments:

```csharp
// Arrange  
// Act  
// Assert
```

The crucial thing here is that the `Act` stage is exactly one statement. That one statement is nothing more than a call to the one method that you are trying to test. Keeping that one statement as simple as possible is also very important. For example, this is not ideal:

```csharp
int result = myObj.CallSomeMethod(GetComplexParam1(), GetComplexParam2(), GetComplexParam3());
```

This style is not recommended because way too many things can go wrong in this one statement. All the `GetComplexParamN()` calls can throw for a variety of reasons unrelated to the test itself. It is thus unclear to someone running into a problem why the failure occurred.

The ideal pattern is to move the complex parameter building into the `Arrange` section:

```csharp
// Arrange
P1 p1 = GetComplexParam1();
P2 p2 = GetComplexParam2();
P3 p3 = GetComplexParam3();

// Act
int result = myObj.CallSomeMethod(p1, p2, p3);

// Assert
Assert.AreEqual(1234, result);
```

Now the only reason the line with `CallSomeMethod()` can fail is if the method itself blew up. This is especially important when you're using helpers such as `ExceptionHelper`, where the delegate you pass into it must fail for exactly one reason.


### Testing exception messages

In general testing the specific exception message in a unit test is important. This ensures that the exact desired exception is what is being tested rather than a different exception of the same type. In order to verify the exact exception it is important to verify the message.

To make writing unit tests easier it is recommended to compare the error message to the RESX resource. However, comparing against a string literal is also permitted.

```csharp
var ex = Assert.Throws<InvalidOperationException>(() => fruitBasket.GetBananaById(1234));
Assert.Equal(Strings.FormatInvalidBananaID(1234), ex.Message);
```

### Use xUnit.net's plethora of built-in assertions

xUnit.net includes many kinds of assertions – please use the most appropriate one for your test. This will make the tests a lot more readable and also allow the test runner report the best possible errors (whether it's local or the CI machine). For example:

!!! warning "The following are incorrect:"

```csharp
Assert.Equal(true, someBool);
Assert.True("abc123" == someString);
Assert.True(list1.Length == list2.Length);

for (int i = 0; i < list1.Length; i++)
{
    Assert.True(String.Equals, list1[i], list2[i], StringComparison.OrdinalIgnoreCase));
}
```
!!! note "They are good:"

```csharp
Assert.True(someBool);
Assert.Equal("abc123", someString);

// built-in collection assertions!
Assert.Equal(list1, list2, StringComparer.OrdinalIgnoreCase);
```

### Parallel tests

By default all unit test assemblies should run in parallel mode, which is the default. Unit tests shouldn't depend on any shared state, and so should generally be runnable in parallel. If the tests fail in parallel, the first thing to do is to figure out *why*; do not just disable parallel tests!

For functional tests it is reasonable to disable parallel tests.


## Solution and project folder structure and naming

Solution files go in the repo root.

Solution names match repo names (e.g. Mvc.sln in the Mvc repo).

Every project also needs a `project.json` and a matching `.xproj` file. This `project.json` is the source of truth for a project's dependencies and configuration options.

Solutions need to contain solution folders that match the physical folders (`src`, `test`, etc.).

For example, in the `Fruit` repo with the `Banana` and `Lychee` projects you would have these files checked in:

```
/Fruit.sln
/src
/src/Fruit.Banana
/src/Fruit.Banana/project.json
/src/Fruit.Banana/Banana.kproj
/src/Fruit.Banana/Banana.cs
/src/Fruit.Banana/Util/BananaUtil.cs
/src/Fruit.Lychee
/src/Fruit.Lychee/project.json
/src/Fruit.Lychee/Lychee.kproj
/src/Fruit.Lychee/Lychee.cs
/src/Fruit.Lychee/Util/LycheeUtil.cs
/test
/test/Fruit.Banana.Tests
/test/Fruit.Banana.Tests/project.json
/test/Fruit.Banana.Tests/BananaTest.kproj
/test/Fruit.Banana.Tests/BananaTest.cs
/test/Fruit.Banana.Tests/Util/BananaUtilTest.cs
```

Note that after running the `build` command the system will generate the following files:

```
/build/*.shade
```

All these files are set to be ignored in the `.gitignore` file.
