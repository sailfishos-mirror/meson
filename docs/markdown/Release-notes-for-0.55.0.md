---
title: Release 0.55.0
short-description: Release notes for 0.55.0
...

# New features

## rpath removal now more careful

On Linux-like systems, Meson adds rpath entries to allow running apps
in the build tree, and then removes those build-time-only rpath
entries when installing. Rpath entries may also come in via LDFLAGS
and via .pc files. Meson used to remove those latter rpath entries by
accident, but is now more careful.

## Added ability to specify targets in `meson compile`

It's now possible to specify targets in `meson compile`, which will
result in building only the requested targets.

Usage: `meson compile [TARGET [TARGET...]]`
`TARGET` has the following syntax: `[PATH/]NAME[:TYPE]`.
`NAME`: name of the target from `meson.build` (e.g. `foo` from `executable('foo', ...)`).
`PATH`: path to the target relative to the root `meson.build` file. Note: relative path for a target specified in the root `meson.build` is `./`.
`TYPE`: type of the target (e.g. `shared_library`, `executable` and etc)

`PATH` and/or `TYPE` can be omitted if the resulting `TARGET` can be used to uniquely identify the target in `meson.build`.

For example targets from the following code:
```meson
shared_library('foo', ...)
static_library('foo', ...)
executable('bar', ...)
```
can be invoked with `meson compile foo:shared_library foo:static_library bar`.

## Test protocol for gtest

Due to the popularity of Gtest (google test) among C and C++
developers Meson now supports a special protocol for gtest. With this
protocol Meson injects arguments to gtests to output JUnit, reads that
JUnit, and adds the output to the JUnit it generates.

## meson.add_*_script methods accept new types

All three (`add_install_script`, `add_dist_script`, and
`add_postconf_script`) now accept ExternalPrograms (as returned by
`find_program`), Files, and the output of `configure_file`. The dist and
postconf methods cannot accept other types because of when they are run.
While dist could, in theory, take other dependencies, it would require more
extensive changes, particularly to the backend.

```meson
meson.add_install_script(find_program('foo'), files('bar'))
meson.add_dist_script(find_program('foo'), files('bar'))
meson.add_postconf_script(find_program('foo'), files('bar'))
```

The install script variant is also able to accept custom_targets,
custom_target indexes, and build targets (executables, libraries), and
can use built executables a the script to run

```meson
installer = executable('installer', ...)
meson.add_install_script(installer, ...)
meson.add_install_script('foo.py', installer)
```

## Machine file constants

Native and cross files now support string and list concatenation using
the `+` operator, and joining paths using the `/` operator. Entries
defined in the `[constants]` section can be used in any other section.
An entry defined in any other section can be used only within that
same section and only after it has been defined.

```ini
[constants]
toolchain = '/toolchain'
common_flags = ['--sysroot=' + toolchain + '/sysroot']

[properties]
c_args = common_flags + ['-DSOMETHING']
cpp_args = c_args + ['-DSOMETHING_ELSE']

[binaries]
c = toolchain + '/gcc'
```

## Configure CMake subprojects with Meson.subproject_options

Meson now supports passing configuration options to CMake and
overriding certain build details extracted from the CMake subproject.

The new CMake configuration options object is very similar to the
[[@cfg_data]] object returned
by [[configuration_data]]. It
is generated by the `subproject_options` function

All configuration options have to be set *before* the subproject is
configured and must be passed to the `subproject` method via the
`options` key. Altering the configuration object won't have any effect
on previous `cmake.subproject` calls.

**Note:** The `cmake_options` kwarg for the `subproject` function is
now deprecated since it is replaced by the new `options` system.

## find_program: Fixes when the program has been overridden by executable

When a program has been overridden by an executable, the returned
object of find_program() had some issues:

```meson
# In a subproject:
exe = executable('foo', ...)
meson.override_find_program('foo', exe)

# In main project:
# The version check was crashing Meson.
prog = find_program('foo', version : '>=1.0')

# This was crashing Meson.
message(prog.path())

# New method to be consistent with built objects.
message(prog.full_path())
```

## Response files enabled on Linux, reined in on Windows

Meson used to always use response files on Windows,
but never on Linux.

It now strikes a happier balance, using them on both platforms,
but only when needed to avoid command line length limits.

## `unstable-kconfig` module renamed to `unstable-keyval`

The `unstable-kconfig` module is now renamed to `unstable-keyval`. We
expect this module to become stable once it has some usage experience,
specifically in the next or the following release


## Fatal warnings in `gnome.generate_gir()`

`gnome.generate_gir()` now has `fatal_warnings` keyword argument to
abort when a warning is produced. This is useful for example in CI
environment where it's important to catch potential issues.

## b_ndebug support for D language compilers

D Language compilers will now set -release/--release/-frelease (depending on
the compiler) when the b_ndebug flag is set.

## Meson test now produces JUnit xml from results

Meson will now generate a JUnit compatible XML file from test results.
it will be in the `meson-logs` directory and is called
`testlog.junit.xml`.

## Config tool based dependencies no longer search PATH for cross compiling

Before 0.55.0 config tool based dependencies (llvm-config,
cups-config, etc), would search system $PATH if they weren't defined
in the cross file. This has been a source of bugs and has been
deprecated. It is now removed, config tool binaries must be specified
in the cross file now or the dependency will not be found.

## Rename has_exe_wrapper -> can_run_host_binaries

The old name was confusing as it didn't really match the behavior of
the function. The old name remains as an alias (the behavior hasn't
changed), but is now deprecated.

## String concatenation in meson_options.txt

It is now possible to use string concatenation (with the `+`
operator) in the `meson_options.txt` file. This allows splitting long
option descriptions.

```meson
option(
  'testoption',
  type : 'string',
  value : 'optval',
  description : 'An option with a very long description' +
                'that does something in a specific context'
)
```

## Wrap fallback URL

Wrap files can now define `source_fallback_url` and
`patch_fallback_url` to be used in case the main server is temporarily
down.

## Clang coverage support

llvm-cov is now used to generate coverage information when clang is
used as the compiler.

## Local wrap source and patch files

It is now possible to use the `patch_filename` and `source_filename`
value in a `.wrap` file without `*_url` to specify a local source /
patch file. All local files must be located in the
`subprojects/packagefiles` directory. The `*_hash` entries are
optional with this setup.

## Local wrap patch directory

Wrap files can now specify `patch_directory` instead of
`patch_filename` in the case overlay files are local. Every files in
that directory, and subdirectories, will be copied to the subproject
directory. This can be used for example to add `meson.build` files to
a project not using Meson build system upstream. The patch directory
must be placed in `subprojects/packagefiles` directory.

## Patch on all wrap types

`patch_*` keys are not limited to `wrap-file` any more, they can be
specified for all wrap types.

## link_language argument added to all targets

Previously the `link_language` argument was only supposed to be
allowed in executables, because the linker used needs to be the linker
for the language that implements the main function. Unfortunately it
didn't work in that case, and, even worse, if it had been implemented
properly it would have worked for *all* targets. In 0.55.0 this
restriction has been removed, and the bug fixed. It now is valid for
`executable` and all derivative of `library`.

## meson dist --no-tests

`meson dist` has a new option `--no-tests` to skip build and tests of
generated packages. It can be used to not waste time for example when
done in CI that already does its own testing.

## Force fallback for

A newly-added `--force-fallback-for` command line option can now be
used to force fallback for specific subprojects.

Example:

```
meson setup builddir/ --force-fallback-for=foo,bar
```

## Implicit dependency fallback

`dependency('foo')` now automatically fallback if the dependency is
not found on the system but a subproject wrap file or directory exists
with the same name.

That means that simply adding `subprojects/foo.wrap` is enough to add
fallback to any `dependency('foo')` call. It is however requires that
the subproject call `meson.override_dependency('foo', foo_dep)` to
specify which dependency object should be used for `foo`.

## Wrap file `provide` section

Wrap files can define the dependencies it provides in the `[provide]`
section. When `foo.wrap` provides the dependency `foo-1.0` any call do
`dependency('foo-1.0')` will automatically fallback to that subproject
even if no `fallback` keyword argument is given. See [Wrap
documentation](Wrap-dependency-system-manual.md#provide_section).

## `find_program()` fallback

When a program cannot be found on the system but a wrap file has its
name in the `[provide]` section, that subproject will be used as
fallback.

## Test scripts are given the exe wrapper if needed

Meson will now set the `MESON_EXE_WRAPPER` as the properly wrapped and
joined representation. For Unix-like OSes this means python's
shelx.join, on Windows an implementation that attempts to properly
quote windows argument is used. This allow wrapper scripts to run test
binaries, instead of just skipping.

for example, if the wrapper is `['emulator', '--script']`, it will be passed
as `MESON_EXE_WRAPPER="emulator --script"`.

## Added ability to specify backend arguments in `meson compile`

It's now possible to specify backend specific arguments in `meson compile`.

Usage: `meson compile [--vs-args=args] [--ninja-args=args]`

```
  --ninja-args NINJA_ARGS    Arguments to pass to `ninja` (applied only on `ninja` backend).
  --vs-args VS_ARGS          Arguments to pass to `msbuild` (applied only on `vs` backend).
```

These arguments use the following syntax:

If you only pass a single string, then it is considered to have all
values separated by commas. Thus invoking the following command:

```
$ meson compile --ninja-args=-n,-d,explain
```

would add `-n`, `-d` and `explain` arguments to ninja invocation.

If you need to have commas or spaces in your string values, then you
need to pass the value with proper shell quoting like this:

```
$ meson compile "--ninja-args=['a,b', 'c d']"
```

## Introspection API changes

dumping the AST (--ast): **new in 0.55.0**
- prints the AST of a meson.build as JSON

## `--backend=vs` now matches `-Db_vscrt=from_buildtype` behaviour in the Ninja backend

When `--buildtype=debugoptimized` is used with the Ninja backend, the
VS CRT option used is `/MD`, which is the [behaviour documented for
all
backends](https://mesonbuild.com/Builtin-options.html#b_vscrt-from_buildtype).
However, the Visual Studio backend was pass `/MT` in that case, which
is inconsistent.

If you need to use the MultiThreaded CRT, you should explicitly pass
`-Db_vscrt=mt`
