# Use casadi easily as a dependency in cmake

This project is a couple of lines in cmake to provide an easy way to use casadi
in your own c++ code without the need to install casadi to your system.

## Installation
include this repository as a submodule into your own project:
```
git submodule add https://github.com/eike-fokken/consumecasadi.git
```

Set it up with

```
git submodule update --init --recursive
```

## Usage
It will provide a library target named `casadi`.

In your `CMakeLists.txt` you can now use it like this:

```
add_subdirectory(consumecasadi)
target_link_libraries(YourBinary casadi)
```

## How it works
It is still a bit hacky. For example the dependencies to be built are hard-coded
(at the moment IPOPT, MUMPS, METIS and HIGHS). The configure step (`cmake -S
<source_dir> -B <build_dir>`) carries out the configure step of casadi and uses
a build directory within the casadi directory itself. The build step (`cmake
--build <build_dir>`) both builds casadi and its dependencies and then installs
them into the build directory of your project (`<build_dir>`). The installed
libraries and headers are then used for the casadi library target.
