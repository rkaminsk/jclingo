# jclingo

This is a small example that creates a low-level binding java to clingo and
calls the `clingo_version()` function and computes a stable model as an
example.

The example requires clingo to be installed. If clingo has been installed in a
non-standard location, `-DClingo_DIR=<location>` can be passed to help cmake
find it. Similarly, the location of the java installation might have to be
specified. For this, `-DJAVA_HOME=<location>` should be used.

# SWIG and Callbacks

SWIG supports callbacks in form of virtual functions in C++ classes. Callbacks
are not supported at all if only C processing is enabled. In the clingo header
there are some enums that have the same name as some functions in the API.
Since SWIP does not prefix enums in C++ mode, compilation fails. For now, it is
easiest to patch the clingo header to be able to use C++ processing.

```bash
sed 's/\(enum clingo_[^ ]*[^ _].\) /\1_e /' <path-to-clingo-header>
```

I will rename the enums in the next clingo release to avoid such clashes.

# Error Handling

The example implements no error handling whatsoever. This is of course
important but also quite involved. Especially, when it comes to callbacks.

## Example Setup on Ubuntu 20.04

Note that there are two cmake configure/build calls below. The first time the
package is build will result in an error; the second call should run through.
This is because swig generates a lot of java files, which would ideally be
listed in `CMakeLists.txt` file. This can wait till the final version of the
package because I expect this list of files to change a lot during development.

```bash
sudo add-apt-repository ppa:potassco/focal-wip
sudo apt-get update
sudo apt install cmake swig openjdk-14-jdk g++ libclingo-dev
sudo sed 's/\(enum clingo_[^ ]*[^ _].\) /\1_e /' /usr/include/clingo.h
cmake -DJAVA_HOME=/usr/lib/jvm/java-14-openjdk-amd64 -B build
cmake --build build
cmake -DJAVA_HOME=/usr/lib/jvm/java-14-openjdk-amd64 -B build
cmake --build build
```

If everything went right, the example can be executed like this:
```bash
➜ java -Djava.library.path=build -jar build/example.jar
version: 5.5.0
```

## Hint for Development on Windows

Both clingo and swig should fully support Windows but setup on Windows might be
more tricky. An easy alternative is to grab [Ubuntu 20.04 from the Microsoft
store][ms-ubuntu] and follow the above instructions.

[ms-ubuntu]: https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71
