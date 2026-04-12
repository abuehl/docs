# Let's bite the Bullet: Module Units shouldn't implicitly import anything

Currently, [the C++ standard](https://eel.is/c++draft/module) states, that
an implementation unit of the following form:

```cpp
// Translation unit #1
module M;
...
```

*implicitly imports* its interface

```cpp
// Translation unit #2
export module M;
...
```

This presents a problem when we want to split the interface into partitions:

```cpp
// Translation unit #3
export module M;

export import :P1;
export import :P2;
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

The problem with this is, that if the partition unit `:P2` is modified,
translation units `#4` and `#5` also need to be recompiled, because they
both implicitly import TU `#3`, even though `:P2` is not imported neither
in TU `#4` nor TU `#5`.

For small toy projects this may be acceptable, but this approach clearly
doesn't scale, because it causes unneeded recompilations.

An alternative is to use "internal partitions"

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

Basically, the sole motivation to use internal partitions in this case is:
Partitions do not implicitly import anything.

There have been discussions to add yet another type of partition, using for
example the follwoing syntax:

```cpp
// Translation unit #8
module M:;
...
```

This would be an internal partition that doesn't have a name (an "anoymous
partition").

Basically this would add a kludge to the standard, on top of a kludge.

Let's face it: The true canonical means to produce implementation files for
modules is to use the `"module"` keyword, followed by the name of the module
(TU `#1`).

We may say that this problem is too small to be of concern.

But this problem reveals a fundamental flaw in the current design of modules:
The current semantic of translation unit `#1` bundles two things together:

1. Signaling an implementation unit of a module
2. Importing the interface of the module

For module interfaces which don't use partitions, this is not a poblem. But
it is a problem for interfaces which are aggregates of partitions. These
can't be separated, because they do have to be exported from the primary module
interface unit, in order to make the exported declrations in the partitions
available to the importers of the module.

The convenience of implicitly getting the declarations from the interface
turns into a significant inconvience if the interface is an aggregate
of partitions.

We might say that partitions are only a marginal use case and these
"inconveniences" do not need to be fixesd. But partitions are an integral
part of the standard and they do serve an important purpose.

## Conclusion

The correct fix for this is to attack the root cause and *stop implicitly
importing the interface* of module implementation units.

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
actually more control over what happens.

This would break existing C++ code, but the alternatives are worse.

Perhaps an additional short-hand could be introduced:

```cpp
// Translation unit #11
module import M;
...
```

which would combine the import and the module declaration on a single
line.

However: Not importing anything should be the new default.
