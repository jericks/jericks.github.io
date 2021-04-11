---
title: "Using GEOS and C++ with Conan and CMAKE"
date: 2021-03-25T20:33:45-07:00
tags: ["geos","c++","conan","cmake"]
featured_image: "img/creek.jpg"
draft: false
---

One of the best ways to get started using the GEOS library with C++ is to use Conan and CMAKE. 

[GEOS](https://trac.osgeo.org/geos) is the Geometry Engine, Open Source library written in C and C++.  Is is a port of the [Java Topology Suite](https://github.com/locationtech/jts) (JTS).  Both libraries include spatial predicate functions and spatials operators.  GEOS is used extensively in the PostGIS extension for Postgres.
[Conan](https://conan.io/) is a C/C++ package manager.  It allows you to declare what libraries your code depends on.
Finally, [CMake](https://cmake.org/) is one of the  most popular ways to build C++ applications.  

<!--more-->

Let's build a GEOS hello world style application. First you need to install [conan](https://docs.conan.io/en/latest/installation.html#install-with-pip-recommended) and [CMAKE](https://cmake.org/install∏P).

Then, create a **conanfile.txt** file. This declares that our code depends on GEOS version 3.8.1. 

```
[requires]
geos/3.8.1

[generators]
cmake_find_package
```

After that, create a **CMakeLists.txt** file.  This basically says that we require CMAKE 3.1 or higher, we need C++11, and that
we are building an executable program using a single file called main.cpp that depends on the GEOS library.

```
project(geos)
cmake_minimum_required(VERSION 3.1)

set (CMAKE_CXX_STANDARD 11)
set(CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})
message(${CMAKE_BINARY_DIR})

add_executable(app main.cpp)
find_package(geos)
target_link_libraries(app GEOS::GEOS)
```

Now we are ready to create our simple C++ GEOS application.  After declaring a few includes, we read in a WKT string, buffer the resulting geometry by 2, and then 
write the buffered geometry back to WKT and the standard output stream.

```cpp
#include <iostream>
#include <geos/io/WKTReader.h>
#include <geos/io/WKTWriter.h>
#include <geos/geom/Geometry.h>

int main() {

    geos::io::WKTReader reader;
    auto geometry = reader.read("POINT (1 1)");
    
    auto buffer = geometry->buffer(2, 10);

    geos::io::WKTWriter writer;
    auto wkt = writer.write(buffer.get());
    std::cout << wkt << std::endl;
    
    return 0;
}
```

Now we are ready to build. Cross your fingers.

First, create a build directory.

```bash
mkdir build
cd build
```

Then install the GEOS dependecy using conan.

```bash
conan install ..
```

Create the CMake build.

```bash
cmake ..
```

Now build the your application.

```bash
make
```

If this builds correctly, you can now run your simple GEOS application.

```bash
./app
```

You should see the WKT of a Polygon.

```
POLYGON ((3.0000000000000000 1.0000000000000000, 2.9753766811902755 0.6871310699195385, 2.9021130325903073 0.3819660112501058, 2.7820130483767365 0.0920190005209074, 2.6180339887498958 -0.1755705045849452, 2.4142135623730963 -0.4142135623730938, 2.1755705045849485 -0.6180339887498936, 1.9079809994790962 -0.7820130483767345, 1.6180339887498980 -0.9021130325903062, 1.3128689300804655 -0.9753766811902749, 1.0000000000000042 -1.0000000000000000, 0.6871310699195428 -0.9753766811902762, 0.3819660112501100 -0.9021130325903086, 0.0920190005209113 -0.7820130483767382, -0.1755705045849418 -0.6180339887498982, -0.4142135623730911 -0.4142135623730989, -0.6180339887498916 -0.1755705045849507, -0.7820130483767331 0.0920190005209015, -0.9021130325903055 0.3819660112500999, -0.9753766811902747 0.6871310699195328, -1.0000000000000000 0.9999999999999944, -0.9753766811902764 1.3128689300804561, -0.9021130325903088 1.6180339887498896, -0.7820130483767382 1.9079809994790886, -0.6180339887498982 2.1755705045849418, -0.4142135623730989 2.4142135623730914, -0.1755705045849509 2.6180339887498913, 0.0920190005209014 2.7820130483767329, 0.3819660112500998 2.9021130325903055, 0.6871310699195325 2.9753766811902747, 0.9999999999999943 3.0000000000000000, 1.3128689300804561 2.9753766811902764, 1.6180339887498896 2.9021130325903091, 1.9079809994790886 2.7820130483767382, 2.1755705045849414 2.6180339887498985, 2.4142135623730909 2.4142135623730994, 2.6180339887498913 2.1755705045849512, 2.7820130483767329 1.9079809994790988, 2.9021130325903055 1.6180339887499002, 2.9753766811902747 1.3128689300804675, 3.0000000000000000 1.0000000000000000))
```