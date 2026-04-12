# C++ Modules: Module Units shouldn't implicitly import anything

Currently, the C++ standard states, that an implementation unit of module M

```cpp
module M;
...
```

implicitly imports its interface

```cpp
export module M;
...
```
