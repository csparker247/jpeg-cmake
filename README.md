# IJG's libjpeg (with CMake)
Current Version: `libjpeg 9c (9.3.0)`

## Usage
This project is meant to provide drop-in CMake support for the IJG's JPEG
library. Simply run CMake as normal:

```Shell
mkdir build/
cd build/
cmake ..
make
```

Alternatively, the important CMake files can be copied to any `libjpeg`
source directory:
```Shell
cp resources/* ~/jpeg-9c/
cd ~/jpeg-9c/
mkdir build/
cd build/
cmake ..
make
```

## Updating libjpeg
```Shell
# Delete the contents of the libjpeg subdirectory
cd libjpeg/
rm -rf *

# Copy the source files for libjpeg into the libjpeg subdirectory
cp -a ~/jpeg-9c/ .

# Rerun the CMake build process
cd ../build/
cmake .
make
```

## Advanced Options
### Shared and Static Libraries
`jpeg-cmake` emulates the behavior of `libjpeg` and compiles both static and
shared libraries by default. Selective compilation of shared and static
libraries can be controlled with the `BUILD_*_LIBS` CMake flags:

```Shell
cmake -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=ON ..
```

### Extra Components
The `libjpeg` utility programs are built by default. To disable compilation of
these tools, set the `BUILD_EXECUTABLES` flag:
```Shell
cmake -DBUILD_EXECUTABLES=OFF ..
```

__(COMING SOON)__ The `libjpeg` tests are built by default. To disable, set the
`BUILD_TESTS` flag:
```Shell
cmake -DBUILD_TESTS=OFF ..
```

By default, executables and tests are linked against the shared libraries. If
static libraries are enabled, executables can optionally be linked statically
by using the `LINK_STATIC` flag:
```Shell
cmake -DLINK_STATIC=ON ..
```
