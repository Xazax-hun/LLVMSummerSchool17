# LLVM Summer School 2017 Exercises

You can use the following build script to set up your development
environment: https://goo.gl/cieeWh

Note that, if you are working on lab machines the script is already run
for you. 

In this repository, you will find the instructions and also some 
patch files that you need to apply on top of Clang 4.0.1 in order
to be able to get started. 

Enjoy! 

# The Task

In C++ it is undefined behavior to assign an out of range element to
an enum. 

```c++
enum A { a, b, c };

void f() {
  A a = (A)5; // Undefined!
}

```

During the exercises, we will develop multiple methods to catch such
issues using static and dynamic analysis. The point of these
exercises to get to know these tools and the advantages/disadvatages of
static vs dynamic approach to a problem. 

# Static Analysis Using Clang Tidy

In this task, you should write a clang tidy checker that warns when an out
of range constant is assigned to an enum variable.
Note that you should also warn when such constant is passed as an enum 
argument!

During the lectures, you already learned about ASTMatchers and how a tidy
check looks like. You might, however, not be familiar with the API of
the Clang AST, and how to deal with enums.

There is already a tidy check that deals with enums, you can check it out to
see how to deal with enums:
https://github.com/llvm-mirror/clang-tools-extra/blob/master/clang-tidy/misc/SuspiciousEnumUsageCheck.cpp

To get started you first need to run a script to generate the template of
a new check:
http://clang.llvm.org/extra/clang-tidy/#writing-a-clang-tidy-check

The module should be misc and the name should be enum-out-of-range
```
cd clang-tidy
python2 add_new_check.py misc enum-out-of-range
```

After generating the template for the new check, you should apply 
clang-tools-extra.patch to get the test cases for your assignment. 

To apply the patch go to clang-tools-extra source directory:

```
patch -p1 < clang-tools-extra.patch
```

You can run the test-cases using:

```
ninja check-clang-tools
```
Hint: to get started look at the AST dump and the ASTMatcher reference. 


## Optional Extension

Clang Tidy is not powerful enough to reason about the possible values of
variables. You can use Clang Tidy to transform source code! 
In case a variable is assigned to an enum, insert a runtime check whether
that value is in bound! 

For example, transform the following code:

```c++
enum A { a, b, c };

void f(int i) {
  A a = (A)i; 
}
```

To:

```c++
enum A { a, b, c };

void f(int i) {
  A a = (A)((i >= 0 && i < 3)?i:( __builtin_trap(),0)); 
}
```

How to transform headers when included in multiple files? What about macros?

# Static Analysis using Clang Static Analyzer

The Clang Static Analyzer using a powerful symbolic execution engine to
reason about the values of the variables. Use the Clang Static Analyzer to
develop a check for enum out of bound assignments!

Apply clang.patch on top of Clang to get the template for the check and fill
in the stubs.

When you are in the clang source directory:

```
patch -p1 < clang.patch
```

You can run tests the following way:
```
ninja check-clang
```

# Dynamic Analysis

Static Analyzer is not guaranteed to find all the problems and might also
give you false positives. Write an instrumentation pass to detect out of
range assignments for enums at runtime.

More details here: https://github.com/hfinkel/llvm-ss-2017
