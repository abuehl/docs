# Recommendations for Using Modules

### Prefer small modules

I've tried using large modules, but I haven't see faster builds when doing
full builds with large modules. The build time for full build stays roughly
the same with more, but smaller modules.

More but smaller modules provider faster rubilds when something changes.

### Only use partitions if you really must

Partitions provide the advantage, that you can forward delcare classes
across partition boundaries. This isn't possible with modules. If you
have a pointer or a reference to an object delcared in module M,
you need to import M.

However, usage of partitions should be limited to cases where you
really need them. If you must, keep the resulting module small.

Large modules with a large number of partitions cause a lot of
recompilations, of an inteterface partition changes.

### Avoid using internal partitions

Prefer using modules. Compose a larger module of smaller ones.
Don't try to use internal partitions for hiding purposes. It's no
problem to create a module which is only intended for internal
use.

(last edited: 2026-04-25)
