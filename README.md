<!---
{
  "id": "3d0ad147-62b6-44c1-8263-ad4d8340422a",
  "depends_on": ["5ae3c504-9947-49d3-9dd5-1ac6f9b22a7b"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-12-04",
  "keywords": ["C++", "CMake", "Linux", "Build systems", "Compilation"]
}
--->

# Compiling a Single `.cpp` File Into an Executable Using CMake on Linux

> In this exercise you will learn how to create a minimal CMake-based build configuration for a single C++ source file. Furthermore we will explore how CMake generates build scripts on Linux and how to invoke these scripts to obtain a standalone executable.

## Introduction

Modern C++ development increasingly relies on build systems that automate compilation, dependency resolution, and platform-independent configuration. One of the most widely used build systems in contemporary software engineering is **CMake**, a meta-build generator that produces platform-appropriate build files from a declarative project description. While CMake is indispensable for large multi-module codebases, it is equally useful for small or single-file programs because it provides consistency, reproducibility, and scalability. Understanding the workflow for compiling even a single `.cpp` file using CMake is an excellent foundation for later exercises that involve more complex dependency graphs, linking external libraries, or integrating testing frameworks.

In Linux environments, CMake typically cooperates with `make` by generating a `Makefile` in a dedicated build directory. The user then triggers compilation using standard build commands such as `make` or `cmake --build`. This exercise will walk you through every step: writing a simple C++ program, creating a minimal `CMakeLists.txt`, configuring the build, inspecting generated files, compiling the executable, and finally running it. We will also introduce the directory conventions recommended by CMake, such as keeping source files and build artifacts separate to maintain a clean and reproducible workflow.

The goal is not only to compile a program, but also to internalize how CMake interprets project metadata, detects compilers, resolves configuration variables, and emits the required build scripts. This understanding is essential for scaling up to more advanced projects and integrating additional tooling later.

### Further Readings and Other Sources
- CMake documentation: https://cmake.org/cmake/help/latest/
- “Professional CMake: A Practical Guide” (Craig Scott), ISBN 978-1989015017  
- YouTube: *CMake Tutorial For Beginners* by The Cherno  
- DOI: 10.1145/3386162 — Overview of modern C++ toolchains  

---

## Tasks

### 0. Install CMake

Before starting, ensure that CMake is installed on your Linux system. On Ubuntu/Debian-based distributions, use the package manager:

```bash
sudo apt update
sudo apt install cmake
```

Verify the installation by checking the CMake version:

```bash
cmake --version
```

You should see output similar to:
```
cmake version 3.16.3
CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

**Note**: This tutorial requires CMake 3.10 or later (as specified in our `CMakeLists.txt`). Most modern Linux distributions provide a compatible version through their package managers. If you need a newer version than what's available in your distribution's repositories, you can download it directly from the [CMake website](https://cmake.org/download/) or use alternative installation methods like `snap install cmake --classic`.

You'll also need a C++ compiler. On most Linux systems, this is already installed, but you can ensure it's available with:

```bash
sudo apt install build-essential
```

### 1. Create a project directory

Open a terminal and create a clean workspace:

```bash
mkdir cpp_singlefile_cmake
cd cpp_singlefile_cmake
```

Inside this directory, create a subdirectory for source files:

```bash
mkdir src
```

### 2. Write the C++ source file

Create `src/main.cpp` using your preferred editor (recommended: `vim`):

```cpp
#include <iostream>

int main() {
    std::cout << "Hello from CMake single-file build!" << std::endl;
    return 0;
}
```

Save and exit.

### 3. Create the minimal `CMakeLists.txt`

In the project root (`cpp_singlefile_cmake/`), create the file:

```cmake
cmake_minimum_required(VERSION 3.10)

project(SingleFileDemo VERSION 1.0 LANGUAGES CXX)

add_executable(singlefile src/main.cpp)
```

This file declares:

* a required CMake version,
* a project with language `CXX`,
* a single executable target built from one source file.

### 4. Configure the project using CMake

Create a dedicated build directory:

```bash
cmake -B build
```

The `cmake -B build` command tells CMake to create and use a directory named `build` as the **build directory** (also called the "binary directory"). The `-B` flag specifies where CMake should place all generated files, including build scripts, object files, and the final executable.

**Why we want CMake to generate a separate "build" directory:**

1. **Source-Build Separation**: This follows CMake's recommended practice of keeping source files and build artifacts completely separate. Your source code remains in the project root and `src/` directory, while all generated files go into `build/`.

2. **Clean Workspace**: Without a dedicated build directory, CMake would scatter generated files (`Makefile`, `CMakeCache.txt`, `CMakeFiles/`, object files, executables) throughout your source directory, making it cluttered and hard to manage.

3. **Easy Cleanup**: You can completely remove all build artifacts by simply deleting the `build/` directory (`rm -rf build/`) without affecting your source code. This is invaluable when you need a fresh build or want to clean up your project.

4. **Multiple Build Configurations**: You can create multiple build directories (e.g., `build-debug/`, `build-release/`, `build-testing/`) for different configurations while keeping the same source tree.

5. **Version Control Hygiene**: You can easily exclude all build artifacts from version control by adding `build/` to your `.gitignore`, ensuring only source code is tracked.

6. **Reproducibility**: Other developers can clone your repository and immediately run `cmake -B build` to create their own clean build environment without conflicts.

During this step, CMake will:
- Create the `build/` directory if it doesn't exist
- Detect your system's C++ compiler and build tools
- Generate a `Makefile` (on Linux) inside the `build/` directory
- Create a `CMakeCache.txt` file to store configuration variables
- Set up the `CMakeFiles/` subdirectory with internal build metadata

Intermediate output will display detected compilers and the generation of a `Makefile`. If successful, the last lines should indicate that CMake has configured and generated the project.

### 5. Build the executable

```bash
cmake --build ./build/
```

The `cmake --build` command is CMake's platform-independent way to invoke the underlying build system that was generated during the configuration step. Instead of directly calling `make` (which is Linux-specific), this command abstracts the build process across different platforms and generators.

Here's what happens when you run `cmake --build ./build/`:

1. **Build Directory Target**: The `./build/` argument specifies the directory containing the generated build files (Makefile, project files, etc.)
2. **Generator Invocation**: CMake automatically detects the appropriate build tool to use (in this case, `make` on Linux) and invokes it with the correct parameters
3. **Compilation Process**: The underlying build system compiles `src/main.cpp` using the detected C++ compiler with the configuration specified in your `CMakeLists.txt`
4. **Executable Creation**: The compilation produces an executable named `singlefile` (as defined by the `add_executable()` directive) in the `build/` directory

This approach provides several advantages over directly running `make`:
- **Cross-platform compatibility**: The same command works on Windows (MSBuild), macOS (Xcode), and Linux (Make)
- **Generator independence**: Works regardless of whether CMake generated Makefiles, Ninja files, or IDE project files
- **Consistent interface**: Provides uniform build semantics across different development environments

This compiles `main.cpp` and creates an executable named `singlefile` in the build directory.

### 6. Run the program

Execute your final binary: 

```bash
./build/singlefile
```

The expected output is:

```
Hello from CMake single-file build!
```

### 7. (Optional) Inspect generated artifacts

List the directory contents:

```bash
ls -l ./build/
```

You will see:

* `Makefile`
* `CMakeFiles/` directory
* `singlefile` executable
  These illustrate how CMake layers its generated build system around your project metadata.

---

## Questions

Test your comprehension of the CMake workflow with these questions:

**1. What would happen if you ran `cmake --build ./build/` before running `cmake -B build`?**

<details>
<summary>Click to reveal answer</summary>

The build command would fail because the build directory and its generated files (like the Makefile) don't exist yet. CMake needs to configure the project first to create the build system files before it can invoke the build process.

</details>

**2. Why do we put the C++ source file in a `src/` subdirectory instead of directly in the project root?**

<details>
<summary>Click to reveal answer</summary>

Using a `src/` directory follows standard project organization conventions and separates source code from configuration files (like CMakeLists.txt), documentation, and build artifacts. This structure scales well as projects grow and makes the project more maintainable and professional.

</details>

**3. What is the difference between running `make` directly in the build directory versus using `cmake --build ./build/`?**

<details>
<summary>Click to reveal answer</summary>

Both commands achieve the same result in this Linux example, but `cmake --build` is platform-independent and works with any generator (Make, Ninja, Visual Studio, Xcode), while `make` only works when CMake generates Makefiles. Using `cmake --build` makes your workflow portable across different operating systems and build tools.

</details>

**4. If you wanted to completely start over with a clean build, what would be the most efficient approach?**

<details>
<summary>Click to reveal answer</summary>

Delete the entire build directory (`rm -rf build/`) and then reconfigure (`cmake -B build`). This removes all cached configuration, generated files, and compiled objects, ensuring a completely fresh build without affecting your source code.

</details>

**5. What does the `add_executable(singlefile src/main.cpp)` line in CMakeLists.txt actually do?**

<details>
<summary>Click to reveal answer</summary>

This line tells CMake to create a build target called "singlefile" that compiles the source file `src/main.cpp` into an executable. The first argument (`singlefile`) becomes the name of the resulting executable file, and the second argument specifies which source file(s) to compile.

</details>

---

## Advice

When learning CMake, consistency and repetition are essential. Build the habit of separating source and build directories early, because larger projects will rely heavily on this structure for clarity and reproducibility. Always inspect your `CMakeLists.txt` after changes and rerun the configuration step to ensure that CMake properly regenerates the build system. If something does not compile as expected, remove your build directory and reconfigure from scratch—this often resolves stale state issues. As you proceed to more advanced sheets that introduce libraries, multiple targets, or testing frameworks, the workflow you practiced here will remain fundamentally the same. Think of this sheet as laying the groundwork for later exercises where project structure and target properties become more complex. Continue experimenting, adding small variations, and verifying how CMake reacts; the more you do this, the more natural the build process will feel.

