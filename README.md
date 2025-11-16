# Overview

Based on CMake official [tutorial](https://cmake.org/cmake/help/v3.30/guide/tutorial/index.html) that uses Newton's method for finding roots.

# Formatter

If you want to format the `CMakeLists.txt` file, you can follow these steps:

1. First, use pip to install `cmake_format`.

  ```shell
  python -m pip install cmake_format
  ```

2. Then install the "cmake-format" extension from VS Code.

# Configuring the preset

You can generate the `CMakePresets.json` file using the CMake extension in VSCode and add a "configurePreset".

```json
{
  "version": 8,
  "configurePresets": [
    {
      "name": "default",
      "displayName": "Default Configure",
      "binaryDir": "${sourceDir}/out/build/${presetName}",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Debug",
        "CMAKE_INSTALL_PREFIX": "${sourceDir}/out/install/${presetName}"
      }
    }
  ]
}
```

The `CMakeUserPresets.json` file can specify custom local build details, sush as complier paths or build tools. This file should not  be checked into VCS. For example:

```json
{
  "version": 8,
  "configurePresets": [
    {
      "name": "windows-only",
      "inherits": "default",
      "generator": "MinGW Makefiles",
      "cacheVariables": {
        "CMAKE_C_COMPILER": "D:/msys64/ucrt64/bin/gcc.exe",
        "CMAKE_CXX_COMPILER": "D:/msys64/ucrt64/bin/g++.exe",
        "CMAKE_MAKE_PROGRAM": "D:/msys64/ucrt64/bin/mingw32-make.exe"
      }
    }
  ]
}
```

> [!NOTE]
>
> The `CMakeUserPresets.json` file can implicitly includs the `CMakePresets.json` file, even with no `include` field. You can specify the `inherits` field to inherit all fileds from anothor preset.In this example, the "windows-only" preset inherits from the "default" preset defined in `CMakePresets.json`.

# Build project

Excute the following command in the command line:

1. Generate project Buildsystem.

   ```shell
   cmake --preset=default
   # on Windows
   cmake --preset=windows-only
   ```

2. Build project.

   ```shell
   cmake --build --preset=default
   ```

> [!TIP]
>
> Also you can add "buildPresets" preset in the `CMakeUserPresets.json` file.

# Test

Excute the following command in the command line:

```shell
ctest --preset=default
```

If you want to display our CTest results with CDash, navigate to the build directory and run:

```shell
ctest -D Experimental
```

> [!TIP]
>
> You can use "testPresets" preset in the `CMakeUserPresets.json` file.

# Install

Excute the command `cmake --install <dir>`, where `<dir>` corresponds to the `binaryDir` field configured in the "configurePresets" preset. The CMake variable `CMAKE_INSTALL_PREFIX` is used to determine the root of where the files will be installed. In this case, you can run the command as follows:

```shell
cmake --install out/build/default
```

Afterwards, you can check the directory specified by the `CMAKE_INSTALL_PREFIX` variable to verify the installation location.

# Package

To build a binary distribution, from the binary directory run:

```shell
cpack
```

To create an archive of the full source tree you would type:

```shell
cpack --config CPackSourceConfig.cmake
```
