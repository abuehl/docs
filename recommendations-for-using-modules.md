# Recommendations for Using C++ Modules

### Prefer small modules

I've tried using large modules, but I haven't seen faster builds when doing
full builds with large modules. The build time for a full build stays roughly
the same with more, but smaller modules.

More but smaller modules provide faster rebuilds when something changes,
because a smaller number of dependent translation units is affected.

Don't be afraid of occasionally having a module with just one or two classes.
That's fine.

### Only use partitions if you really must

Partitions provide the advantage, that you can forward declare classes
across partition boundaries. This isn't possible with modules. If you
have a pointer or a reference to a class declared in module M,
you need to import M.

However, usage of partitions should be limited to cases where you
really need them. If you must use them, keep the resulting module small.

Large modules with a large number of partitions cause a lot of
recompilations, if an interface partition changes. Because implementation
units ("module M;") implicitly import the whole aggregated interface
of the module.

Don't use the
[internal partition anti-pattern](https://abuehl.github.io/2026/04/22/an-anti-pattern-for-modules.html).

Instead prefer to compose modules of smaller ones.

### Avoid using internal partitions

Prefer using modules instead. Compose larger modules of smaller ones.
Don't try to use internal partitions for hiding purposes. It's no
problem to create a module which is only intended for internal
use.

(last edited: 2026-04-25)
