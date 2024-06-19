---
title: "Safer C programming"
date: "2017-08-11"
---

TL;DR - check out [elfs-clang-plugins, cool plugins for clang](https://github.com/sinelaw/elfs-clang-plugins) made at [elastifile](https://www.elastifile.com/).

Have you ever made the mistake of returning a bool instead of an enum?

```
enum Result do_something(void) {
    ...
    return true;
}
```

In C that's valid (in C++ you can use 'class enum' to avoid it, but if you didn't you'd have the same problem).

No compiler that I know of warns about this C code. One of our newly-open-sourced clang plugins, flags this (and many other) enum-related mistakes:

```
clang -Xclang -load -Xclang ./clang_plugins.so \
      -Xclang -add-plugin -Xclang enums_conversion \
      -c /tmp/test.c
/tmp/test.c:7:12: error: enum conversion to or from enum Result
    return true;
           ^
1 error generated.
```

The package includes:

- enums\_conversion: Finds implicit casts to/from enums and integral types
- include\_cleaner: Finds unused #includes
- large\_assignment: Finds large copies in assignments and initializations (size is configurable)
- private: Prevents access to fields of structs that are defined in private.h files

More information at https://github.com/sinelaw/elfs-clang-plugins

Because C is... not so safe.
