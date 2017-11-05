Snowman: Build Instructions
===========================

Yegor Derevenets <yegor.derevenets@gmail.com>

Prerequisites
-------------

Snowman is written in C++. So, you will need a C++ compiler to
build it. The following compilers are known to work:

* [GCC](http://gcc.gnu.org/) >= 4.6
* [Clang](http://llvm.clang.org) >= 3.1
* [MinGW GCC](http://mingw.org/) >= 4.6
* [MinGW-w64 GCC](http://mingw-w64.sourceforge.net/) >= 4.6
* [Microsoft Visual Studio](http://www.microsoft.com/) >= 2010

The following software packages must be installed before Snowman can be
built:

* [CMake](http://cmake.org/) >= 2.8
* [Boost](http://www.boost.org/) >= 1.49 (headers only)
* [Qt](http://qt-project.org/) >= 4.7

For installation instructions for the above packages, please refer to
the respective web pages and/or your operating system's manual.

> If you want to use Visual Studio 2012 to build applications that would
> run on Windows XP, you have to install Visual Studio 2012 Update 1 or
> later and at [configuration step](#configuration-step) choose `v110_xp`
> toolset in CMake using `-T` switch (support since CMake 2.8.11, known to
> work with `Visual Studio` generators).

### Notes on Building IDA Plug-In

1. The following additional package is required to build IDA plug-in:

   * [IDA SDK](http://www.hex-rays.com/products/ida/) >= 6.1

2. You must use the same compiler as the one used by IDA and the same
version of Qt (different patch levels of the same major.minor version
are binary compatible and work too):

   * IDA 6.1 — MSVC 2010 — Qt 4.7.2.
   * IDA 6.3 — MSVC 2010 — Qt 4.8.1.
   * IDA 6.4 — MSVC 2010 — Qt 4.8.1.
   * IDA 6.5 — MSVC 2010 — Qt 4.8.4.
   * IDA 6.95 — MSVC 2015 — Qt 5.6.0.

3. Qt must be built in namespace `QT`, e.g.:

   `configure.exe -qtnamespace QT -platform win32-msvc2010 -debug-and-release -opensource -no-qt3support -no-multimedia -no-audio-backend -no-phonon -no-webkit -no-script -no-scripttools -nomake demos -nomake examples && nmake`

   or, on Linux:

   `CXX=g++ CC=gcc ./configure -qtnamespace QT -platform linux-g++-32 -debug-and-release -confirm-license -opensource -no-qt3support -no-multimedia -no-audio-backend -no-phonon -no-webkit -no-script -no-scripttools -nomake demos -nomake examples -prefix /home/yegor/opt/qt-4.8.4-32 && make && make install`

4. You must must build decompiler in `Release` mode, because this is the
one used by IDA. Otherwise, you will end up linked against debug Qt
library, while IDA uses the release version. Nothing good comes out of
this.


Building
--------

Snowman is built using a tool called CMake. The build process consists
of two steps: [configuration](#configuration-step) and
[build](#build-step).


### Setting up the Environment ###

You can help your system to find CMake and CMake to find the compiler
and required libraries at non-standard locations by setting the
following environment variables. Alternatively, you may specify these
locations at [configuration step](#configuration-step).

1. Add CMake and the compiler to your `PATH`.
2. Set `BOOST_ROOT` to the path to Boost installation directory (it
   should contain `boost` subdirectory with header files).
3. Set `QTDIR` to the path to the Qt installation directory (it
   should contain `include` and `lib` subdirectories).
4. Set `IDA_SDK_DIR` to the IDA SDK installation directory (it
   should contain `include` and `lib`) subdirectories.


<a name="configuration-step"></a>
### Configuration Step

CMake authors are advocates of out-of-tree builds, so a separate
directory for compilation results must be created:

```
cd /path/to/decompiler && mkdir build && cd build && cmake ../src
```

You can specify a toolchain using CMake's `-G` switch, e.g.:

* `-G "Visual Studio 10"` — generate a Visual Studio 2010 solution
  for building 32-bit application.
* `-G "Visual Studio 11 Win64"` — generate a Visual Studio 2012
  solution for building 64-bit application.
* `-G "MinGW Makefiles"` — generate Makefiles for building by
  MinGW GCC and `mingw32-make`.

You can choose a build mode (`Debug`, `Release`, `RelWithDebInfo`, or
`MinSizeRel`):

* `-D CMAKE_BUILD_TYPE=Release`

If you have built Qt in a namespace, do not forget to specify this,
e.g.:

* `-D QT_NAMESPACE=QT`

You can choose whether to build the IDA plug-in and which one to build:

* `-D IDA_PLUGIN_ENABLED=YES` (or `NO`, defaults to `YES` when IDA
  Pro SDK is found).
* `-D IDA_64_BIT_EA_T=YES` (`YES` to build `.p64` version for
  handling 64-bit code, `NO` for building `.plw` version for
  handling 32-bit code).

You can choose to build 32-bit code on 64-bit Linux machine:

* `-D NC_M32=YES`.

You can choose between Qt5 and Qt4:

* `-D NC_QT5=YES` (`YES` for Qt5, `NO` for Qt4).

You can set the installation prefix:

* `-D CMAKE_INSTALL_PREFIX=/install/prefix`

> The build mode specified at the configuration step will not have any
> effect for builds by Visual Studio. There you can choose the build mode
> using `--config` option of CMake directly at [build](#build-step) and
> [installation](#installation) steps.

> On Windows, when choosing release or debug mode, make sure that Qt has
> been built in this mode too. If it was not, the build may fail, or even
> succeed but produce non-working executables.


<a name="build-step"></a>
### Build Step

```
cmake --build .
```


<a name="installation"></a>
### Installation Step

```
cmake --build . --target install
```

> On Windows, when the decompiler is built with Qt4, this command will
> install to the same directory all non-system `.dll` files on which the
> decompiler executables depend, so that the installation can be
> painlessly moved to any other machine and remain workable.

> When IDA plug-in is enabled, it is the only target which is installed.
> The rationale is that IDA is thread-unsafe, and multithreading is
> disabled in the builds with plug-in enabled. This gives you less chances
> to install single-threaded standalone version of the decompiler (which
> you should not normally want).
