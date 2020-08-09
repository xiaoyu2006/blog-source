---
title: Make wheels with Autotools & C/C++
date: 2020-08-08 22:00:00
---

This how-to guide will teach you how to make wheels with Autotools & C/C++

# Introduction

## Requirements

My Autotools versions are:

- Automake 1.16.1
- Autoconf 2.69
- Libtool 2.4.6

And I'm on OS X. Installation guide will not be included.

## Product

We'll make a simple C lib (C++ compatible) called `libts` helps you to get time between two calls.

# Procedures

Time to actually make something!

## Basic sources

Our project starts simply like this, including a header file and a source file:

```
.
├── ts.c
└── ts.h

0 directories, 2 files
```

And codes are shown below:

`ts.h`
```c
#ifndef __TS_H__
#define __TS_H__

#include <sys/time.h>
#include "config.h" /* This file will be generated later */

/* For C++ compatiblity */
#ifdef __cplusplus
extern "C" {
#endif

#define FIRST_CALL -1.0

/* 
* Returns the time passed in seconds before the latest call.
* If it's the first time called, return FIRST_CALL.
*/
extern double getTimeDuration(void);

/* End of the extern "C" above */
#ifdef __cplusplus
}
#endif

#endif /* __TS_H__ */
```

`ts.c`
```c
#include "ts.h"

double getTimeDuration()
{
    static double latest = 0; /* Last call */
    double sec; /* Current time in second */
    double ret; /* Return value */

    #ifdef HAVE_GETTIMEOFDAY /* This macro comes from config.h */

    /* In some specific OS, gettimeofday() is available */
    /* See https://man7.org/linux/man-pages/man2/gettimeofday.2.html */

    struct timeval tv;
    gettimeofday(&tv, NULL);
    sec = tv.tv_sec;
    sec += tv.tv_usec / 1000000.0;

    #else /* HAVE_GETTIMEOFDAY */

    /* Or we can use time() instead. */

    sec = time(NULL);

    #endif /* HAVE_GETTIMEOFDAY */

    ret = sec - latest; /* Calculate difference */
    latest = sec; /* Update latest */
    if (ret == sec) /* First call, return special value */
        return FIRST_CALL;
    else
        return ret;
}
```

Now the source code is done. Let's install Autotools!

## Autotools

Autotools is a complicated build system. We have to create several files.

*Note: The following commands are run at `.` of the source code.*

### `configure.ac`

`configure.ac` is a file for `Autoconf` to generate an `configure` script. It checks availability (in our example, if `gettimeofday()` and `time()` are available) and generates `Makefile` from `Makefile.in`, which will be generated later.

Let's start with an `autoscan` GNU provided. It scans your code and generates an `configure.ac` automatically. Sadly, the generated file is useless without modifying.

```bash
$ autoscan
```

Now your project will be something like this:

```
.
├── autoscan.log
├── configure.scan
├── ts.c
└── ts.h

0 directories, 4 files
```

The `autoscan.log` can be removed safely. What matters is `configure.scan`. We have to rename it to `configure.ac` first:

```bash
$ rm -f autoscan.log
$ mv configure.scan configure.ac
```

`configure.ac` looks like this:

```bash
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_CONFIG_SRCDIR([ts.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.
AC_CHECK_HEADERS([sys/time.h])

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
AC_CHECK_FUNCS([gettimeofday])

AC_OUTPUT
```

It's actually a piece of [m4](https://www.gnu.org/software/m4/m4.html) language, and all those `AC_XXX` stuff are macros and will be expanded into bash scripts. You can write bash in the `configure.ac` directly as well.

As you can see, it's smart to include `AC_CHECK_FUNCS([gettimeofday])`. This will checks if `gettimeofday` is available. Magic! But, we have to modify it anyway.

```bash
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69]) # 1
AC_INIT([libts], [0.1], [username@example.com]) # 2
AC_CONFIG_SRCDIR([ts.c]) # 3
AC_CONFIG_HEADERS([config.h]) # 4

AM_INIT_AUTOMAKE # Modified 5

# Checks for programs.
AC_PROG_CC # 6

# Checks for libraries.

# Checks for header files.
AC_CHECK_HEADERS([sys/time.h]) # 7

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
AC_CHECK_FUNCS([gettimeofday]) # 8

LT_INIT # Modified 9
AC_CONFIG_FILES([Makefile]) # Modified 10

AC_OUTPUT # 11
```

- 1: Checks the minimal version of `autoconf`.
- 2 & 11: Start and end of every `configure.ac`. It also includes some info for your project.
- 3: Check if the source code exists.
- 4: Generates the configuration header named `config.h`.
- 5: Prepare for generating `Makefile`.
- 6: Determine a C compiler to use.
- 7: Check if header file `sys/time.h` is available.
- 8: Check if function `gettimeofday` is available.
- 9: Initialize Libtool. This will be used later.
- 10: Generate `Makefile` from `Makefile.in`, which will be generated later.

And to generate the `configure` file:

```bash
$ aclocal
$ autoconf
$ autoheader
```

And your project will be something like this:

```
.
├── aclocal.m4
├── autom4te.cache
│   ├── output.0
│   ├── output.1
│   ├── output.2
│   ├── output.3
│   ├── requests
│   ├── traces.0
│   ├── traces.1
│   ├── traces.2
│   └── traces.3
├── config.h.in
├── configure
├── configure.ac
├── ts.c
└── ts.h
```

It has generated a lot, right?

### `Makefile.am`

`Makefile.am` is a file for `automake` to generate the `Makefile.in` mentioned above. Now create a `Makefile.am` and write the following stuffs:

```automake
AUTOMAKE_OPTIONS = foreign
include_HEADERS=ts.h
lib_LTLIBRARIES = libts.la
libts_la_SOURCES=ts.c
```

The build target is `libts.la`, containing the source file `ts.c`, which uses `Libtool` to sustain portability. It's simpler than `configure.ac`. No future explanations provided.

Also, note that `AUTOMAKE_OPTIONS` is set to `foreign`, so it won't force us to create those `NEWS`, `AUTHOR`, `ChangeLog`, etc.

To generate `Makefile.in`, run:

```bash
$ libtoolize # Generate supporting files for Libtool
$ automake --add-missing
```

### Tests

Tests are always needed. Let's do this in Autotools' way. First create `test.c`:

```c
#include "ts.h"
#include <assert.h>
#include <stdio.h>

/* Program exits with 0 means tests has passed */
int main(int args, char *argv[])
{
    double first = getTimeDuration();
    assert(first == FIRST_CALL);
    double second = getTimeDuration();
    assert(second != FIRST_CALL);
    puts("OK");
    return 0;
}
```

You can use modern test frameworks too.

Edit `Makefile.am`

```automake
# Lib
AUTOMAKE_OPTIONS = foreign
include_HEADERS=ts.h
lib_LTLIBRARIES = libts.la
libts_la_SOURCES=ts.c

# Tests
TESTS = checkTS
check_PROGRAMS = checkTS
checkTS_SOURCES = test.c
checkTS_LDFLAGS = libts.la
```

Also, if you don't want to type the following commands again:

```bash
$ autoreconf -i
```

Will you hate me?

### configure & build

Simple! Everything is ready now. Do this as usual:

```bash
$ ./configure
$ make
```

And to install:

```bash
$ sudo make install
```

To test:

```bash
$ make check
```

To make a distribution package:

```bash
$ make dist
```

Whoa, you did that! To use this lib in your own programs, just `#include <ts.h>` and link this library (`-Lts`)!

## Product

This demo's distribution can be found [here](https://github.com/xiaoyu2006/blog-source/tree/master/source/_posts/libts-0.1.tar.gz).
