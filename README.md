# mujoco-simulink-blockset-cmake-buildsystem

## Overview

This repo contains a drop-in CMake buildsystem for the  mujoco-simulink-blockset mantained at https://github.com/mathworks-robotics/mujoco-simulink-blockset .

**If you are just interested in using mujoco-simulink-blockset in MATLAB, please follow the official instructions at https://github.com/mathworks-robotics/mujoco-simulink-blockset#installation-instructions.**

This repo is only useful if for any need to compile mujoco-simulink-blockset against a MuJoCo and GLFW libraries that are already in your system, for example if you are also using MuJoCo in other software that is loaded by MATLAB, and you want to avoid ABI or behaviour incompatibilities.

## Usage

How to use this repo

1. Install MuJoCo and GLFW in your system, and make sure that it can be find by CMake.

2. Clone the repository

~~~
git clone https://github.com/ami-iit/mujoco-simulink-blockset-cmake-buildsystem.git
~~~

3. Build it

~~~
cd mujoco-simulink-blockset-cmake-buildsystem
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=<install-prefix> ..
cmake --build . --config Release
cmake --install . --config Release
~~~

4. Use it

Add the `<install-prefix>/mex/mujoco_simulink_blockset` directory to the MATLAB path.

## License

Materials in this repository are distributed under the following license:

> All software is licensed under the BSD-3-Clause License. See [LICENSE](./LICENSE) file for details.

## FAQ

### How the version is chosen?

The version of this `CMake` project is chosen in accordance of the original project, plus an additional version that describes if several CMake buildsystem
versions were released, for example the version `x.y.t` is the one corresponding to the `x.y` version of mujoco-simulink-blockset, while `t` is the number that can be increased if changes are done to the CMake buildsystem.
