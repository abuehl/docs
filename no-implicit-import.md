# C++ Modules: Module Units shouldn't implicitly import anything

Currently, the C++ standard states, that an implementation unit of module M

```cpp
// Translation unit #1
module M;
...
```

implicitly imports its interface

```cpp
// Translation unit #2
export module M;
...
```

This is a problem when we want to split the interface into partitions

```cpp
// Translation unit #3
export module M;

export import :P1;
export import :P2;
...
```

Assuming we want to separate implementations of functions declared in `:P1`,
we could do

```cpp
// Translation unit #4
module M;
import :P1;
...

// Translation unit #5
module M;
import :P1;
...
```

The problem with this is, that if the partition unit of :P2 is modified,
translation units #4 and #5 also need to be recompiled, because they
both implicitly import TU #3, even though :P2 is not imported neither
in TU #4 nor TU #5. For small toy examples this may be acceptable, but
the approach doesn't scale and results in unneeded recompilations.

An alternative is the use "internal partitions"

```cpp
// Translation unit #6
module M:impl.P1.A;
import :P1;
...

// Translation unit #7
module M:impl.P1.B;
import :P1;
...
```

Note that partitions `:impl.P1.A`  and `:impl.P1.B` aren't imported anywhere. 

Using this pattern avoids unneeded recompilations, but it adds the following
problems:

1. The build creates unused BMI files
2. Each partition needs a separate name

Item #2 imposes an obligation to the programmer to choose and maintain arbitrary
names. Compilers are not required to diagnose accidental name clashes. Readers
of this kind of code aren't strictly immediately aware that those partitions
aren't intended to be imported anywhere.

Basically, internal partitions in this case are only used, because they do
not implicitly import anything.

There have been discussions to add yet another type of partition using for
example the follwoing syntax:

```cpp
// Translation unit #8
module M:;
...
```

The semantic of this would be an internal partition that doens't have a name
(an anoymous partition).

Basically this would add a kludge to the standard, on top of another kludge.

Let's face it: The canonical means to produce implementation files of Modules
is to use the "module" keyword followed by the name of the module (TU #1).

We may say that this problem is too small to be of a concern.

But this problem reveals a fundamental flaw in the current design of modules.
The semantic of translation unit #1 bundles two things together:

1. Signaling an implementation unit of a module
2. Importing the interface of module

For module interfaces which don't use partitions, this presents no poblem. But
it does for interfaces which are aggregates of partitions.

The convenience of implicitly getting the declarations from the interface
turns into an significant inconvience if the interface is an aggregate
of partitions.

We might say that partitions are only a marginal use case and this
inconvenience doesn't need to be fixed. But partitions are an integral
part of modules in the standard and they do serve a purpose.

The correct fix for this to attack the root problem and stop implicitly importing
the interface of module implementation units.

Adding yet another kind of partition is the wrong way to solve it.

## Implications

Users would have to change:

```cpp
// Translation unit #9
module M;
...
```

to

```cpp
// Translation unit #10
module M;
import M;
...
```

On first impression, this looks like a big inconvenience, but it gives users
actually more control.

This would break existing C++ code, but the alternatives are worse.

Perhaps an additional short-hand could be introduced:

```cpp
// Translation unit #11
module import M;
...
```

However, not importing anything should be the new default.
