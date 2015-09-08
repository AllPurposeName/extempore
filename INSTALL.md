# Building Extempore from Source

## Get the Extempore source

```
git clone git@github.com:digego/extempore.git
```

Currently we're on the `llvm-3.7` branch. This is because we're still
trying to fix some Windows bugs. We'll merge things back as soon as
we're able.

```
git checkout llvm-3.7
```

## Get the build-time deps

### Deb/Ubuntu

```
sudo apt-get install git g++ portaudio19-dev libpcre3-dev
```

### OSX

```
brew install pcre portaudio
```

### Windows

The version of **PCRE** available through NuGet is pretty old (8.33 as of
writing this) so it's probably worth building it from source.

The NuGet version of **Portaudio** should be fine, but building it
from source is fine too.

We still need one component of the **Boost** libs on Windows
(specifically the ASIO component for TCP/UDP handling). If you've got
the NuGet command line client installed, you can probably do

```
nuget install boost-vc140 & nuget install boost_system-vc140 & nuget install boost_regex-vc140 & nuget install boost_date_time-vc140
```

It doesn't matter how you get these deps, but the Extempore CMake
build process expects to find them in `libs/win64/lib` (for the
library files) and `libs/win64/include` (for the headers). So you'll
need to (probably manually) move them in there, which is a bit
painful, but it's a one-time process.

### LLVM 3.7

LLVM now uses CMake as well. Grab the
[3.7.0 source tarball](http://llvm.org/releases/download.html#3.7.0),
apply both patches (`llparser.patch` and `mcjit.patch` in `extras/`),
and

```
mkdir cmake-build && cd cmake-build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_TERMINFO=OFF -DLLVM_ENABLE_ZLIB=OFF -DCMAKE_INSTALL_PREFIX=/path/to/extempore/llvm-build ..
```

On Windows, you'll probably want to specify a 64-bit generator e.g.
`-G"Visual Studio 14 2015 Win64"`

## Generate build files with CMake

In your `extempore` directory,

```
mkdir cmake-build && cd cmake-build
cmake -DEXT_LLVM_DIR=/path/to/llvm-build-files ..
```

If you've set the `EXT_LLVM_DIR` environment variable you don't have
to provide it again to CMake.

If you want to work "in-tree" (handy for Extempore developers) then
you can set

```
cmake -DIN_TREE=ON ..
```

This will make the `make install` just install `extempore` into
`/usr/local/bin`, and Extempore will keep using the Extempore source
directory (i.e. the cloned repo) for it's `runtime/`, `libs/`, etc.

### Windows

On **Windows**, you'll need to give CMake a few more details

```
md cmake-build && cd cmake-build
cmake -G"Visual Studio 14 2015 Win64" -DEXT_LLVM_DIR=c:\path\to\llvm-build-files
```

## Build Extempore

### Linux/OSX

CMake will generate a `Makefile` in `cmake-build`, with a few useful
targets:

- `make` will build Extempore
- `make install` will install `extempore` into `/usr/local/bin` and
  the rest of the files (the "Extempore share directory") into
  `/usr/local/share/extempore`
- `make uninstall` will remove the installed files
- `make aod_stdlibs` will ahead-of-time compile the standard library

### Windows

CMake will generate a Visual Studio solution (`.sln`) in
`cmake-build`.  Open it, and build the `extempore` target.