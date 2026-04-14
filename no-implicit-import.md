# Let's bite the Bullet: Module Units shouldn't implicitly import anything

Currently, [the C++ standard](https://eel.is/c++draft/module) states, that
an implementation unit of the following form:

```cpp
// Translation unit #1a
module M;
...
```

[*implicitly imports*](https://eel.is/c++draft/module#unit-8) its interface:

```cpp
// Translation unit #2
export module M;
...
```

## The problem

This presents a problem when we want to split the interface into partitions:

```cpp
// Translation unit #3
export module M;

export import :P1;
export import :P2;
```

Assuming we want to separate implementations of functions declared in `:P1`,
we could do:

```cpp
// Translation unit #4a
module M;
// uses nothing from :P2
...

// Translation unit #5a
module M;
// uses nothing from :P2
...
```

*Note: Translation units `#4a` and `#5a` do not use any declarations from
partition `:P2`*

The problem with this is, that if the partition unit `:P2` is modified,
translation units `#4a` and `#5a` also need to be recompiled, because they
both implicitly import TU `#3`, even though nothing from `:P2` is used neither
in TU `#4a` nor TU `#5a`.

For small enough projects, this may be acceptable, but this approach clearly
doesn't scale, because it causes unneeded recompilations.

Note that it is tempting to write:

```cpp
// Translation unit #4b
module M;
import :P1;
// uses nothing from :P2
...

// Translation unit #5b
module M;
import :P1;
// uses nothing from :P2
...
```

which is not incorrect, but redundant today (`:P1` is already implicitly
imported today).

## A messy alternative

An alternative is to use "internal partitions":

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

*Note: Partitions `:impl.P1.A`  and `:impl.P1.B` aren't imported anywhere.*

Using this pattern avoids unneeded recompilations, but it adds the following
new problems:

1. The build creates unused BMI files
2. Each partition needs a unique name, despite not being used anywhere

The second point imposes an obligation for the programmer, to choose and maintain
arbitrary names. Compilers are not required to diagnose accidental name clashes.
Readers of this kind of code aren't immediately aware that those partitions
aren't intended to be imported anywhere.

So this pattern is tedious to write, hard to verify for correctness and noisy
to read.

Basically, the sole reason to use internal partitions in this case is:
Partitions do not implicitly import anything.


## Yet another partition type?

There have been discussions to add yet another type of partition, using for
example the following new syntax:

```cpp
// Translation unit #8
module M:; // note the colon
...
```

This would be an internal partition that doesn't have a name (an "anoymous
partition").

This would *not* implicitly import the interface of module `M` and allow to
write:

```cpp
// Translation unit #9
module M:;
import :P1;
...

// Translation unit #10
module M:;
import :P1;
...
```

Translation units `#9` and `#10` would not need to be recompiled, if partition
`:P2` is modified. Resorting to the messy internal partition trick (TU #6 and #7)
would no longer be needed.

This new syntax would be an improvement over the status quo, but it would add yet
another partition type to the standard, without fixing the root problem, which is
that `module M;` implicitly imports its interface.

The sole purpose of this new syntax would be to have an implementation unit which
doesn't implicitly import the interface of the module.

Let's face it: The true canonical means to produce implementation files for
modules is to use the `"module"` keyword, followed by the name of the module
(TU `#1a`).

## A fundamental flaw

We might say that the problem presented above is too small to be of
concern. But it reveals a fundamental flaw in the current design of modules.

The current semantics of translation unit `#1a` bundles two things together:

1. Defining an implementation unit of a module
2. Importing the interface of the module

For module interfaces which don't export partitions, this is not a poblem. But
it is a problem for interfaces which do so. These partitions can't be separated,
because they *do* have to be exported from the primary module interface unit,
in order to make the exported declarations available to the importers of the
module.

The convenience of implicitly getting the declarations from the interface
turns into a significant *inconvience* if the interface is an aggregation
of partitions.

## Conclusion

The correct fix for this is to attack the root cause and stop *implicitly*
importing the interface of module implementation units.

## Implications

Users would have to change:

```cpp
// Translation unit #1a
module M;
...
```

to

```cpp
// Translation unit #1b
module M;
import M;
...
```

On first impression, this looks like a big inconvenience, but it gives users
actually more control over what happens.

This would break existing module *implementation* C++ code, but the alternatives
are worse.

Perhaps an additional short-hand syntax could be introduced for convenience:

```cpp
// Translation unit #1c
module import M;
...
```

which would combine the import and the module declaration on a single
line.

However: Not importing anything should be the new default.

Translation unit `#1b` separates concerns: `"module M;"` tells us,
where the definitions that follow are attached to, whereas the separate
import(s) tell us, which declarations we need to implement the functions that
follow. 

(last edited: 2026-04-14)

