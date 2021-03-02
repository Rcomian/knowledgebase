Mumble Murmur Server on Gentoo
================================================================

TL;DR: Enable the `cxx` use flag on the `sys-libs/db` package.

Trying to build the Murmur server with the following dependencies:

* `media-sound/murmur`
* `dev-libs/Ice`
* `sys-libs/db`

Default compilation of `sys-libs/Ice` failed with:

```
iostream.h no such file or directory
```

This was actually an include from another package: `sys-libs/db`

Looking at the file, it's using a non-standard C++ include (the iostream header should not have a .h on the end of it). This include is guarded by a directive - if CXX compatibility is specified, it would use the correct header file.

Looking at the `sys-libs/db` package itself, we see it has the `cxx` use flag on it.

Sure enough, enabling `cxx` globably re-built this package and allowed `Ice` to build correctly.
