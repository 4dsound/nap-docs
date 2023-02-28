Building the 4DSOUND engine from scratch {#build_instructions}
===
Introduction
---

This document is meant as a step by step manual on how to build the 4DSOUND engine from scratch on a blank development machine using source codes. Some general knowledge of Git and C++ development is required to be able to use this document.

The 4DSOUND Engine is built on the Spatial Sound framework, which is in turn built on the NAP framework, which is a low overhead, open source, real-time control and visualization platform. The NAP Framework provides a modular build system that can be extended with custom modules and applications written in C++.

The build process of the 4DSOUND engine consists of the following steps:
- Get the NAP source code from Git.
- Get the Spatial Sound Framework and the 4dsound engine source repository and place them in the appropriate directories within the NAP source.
- Package all the library source codes to a platform specific binary release of the combined frameworks.
- Build The Works Engine app from the platform specific framework package that was built in the previous step.

System requirements and prerequisites
---

#### MacOS

MacOS 10.13 or higher.
Make sure XCode and the command line development tools are installed

#### Windows 10 or 11

Visual Studio 2019 (v142)

#### Git

Make sure Git is installed.

#### Qt

In order to build the NAP framework from source you need to have Qt 5 installed:

- Go to https://qt.io and select Downloads for open source users.
- Download the Qt online installer
- During installation select Custom installation
- Filter on the LTS category to download and install Qt 5.12 for your target platform.

Getting the repositories
---

#### nap

Clone the NAP framework source from GitHub:
https://github.com/4dsound/nap

This is a fork from the NAP framework sources, with a few adjustments in the build system that help including the custom modules and apps for spatial sound in general and 4DSOUND engine in particular.

#### thirdparty

Next to the nap directory clone the thirdparty repository from GitHub:
https://github.com/4dsound/thirdparty

The thirdparty repository contains all code, libraries and software dependencies that are needed to develop, build and run NAP and spatial sound software.

#### napaudioadvanced

This repository is enclosed as a submodule within the nap repository so you don’t need to clone it from GitHub manually. Make sure you have access rights.

https://github.com/stijnvanbeek/napaudioadvanced

The napaudioadvanced repository contains a third party NAP module with the same name, that contains utilities and classes to perform and control more advanced audio synthesis and processing.


#### 4dsound_modules

This repository is also included in the nap repository as a submodule, so there is no need to clone it from Github manually. Make sure you have access rights though.

https://github.com/4dsound/4dsound_modules

This repository contains a collection of modules that are needed to work with spatial sound in nap.

#### 4dsound

This repository contains the project and build target for the 4DSOUND Engine application. It needs to be cloned manually in the apps directory within the nap repository.

https://github.com/4dsound/4dsound

Getting the repositories using the command line
---

Here are step by step instructions to clone all necessary repositories from the command line:

```
git clone https://github.com/4dsound/nap
git clone https://github.com/4dsound/thirdparty
cd nap
git submodule update --init --recursive
cd apps
git clone https://github.com/4dsound/4dsound
```

Building a platform release package of the framework
---

Now that all repositories are present in the right locations, the next step is to build a binary release of the NAP Framework and all external modules that are included. This mainly means that all the required NAP modules are build as dynamic libraries and packaged into a release of the framework including all header files for the modules and the sources for the application project. Using this platform release package the application will be built in the next step.

#### Including the 4DSOUND engine application in the build system

In order for our platform release package to include the necessary targets to build the 4DSOUND engine, we need to include the app in the CMake project. Open the file `CMakeLists.txt` in the root nap directory and make sure to remove the “#” sign in front of the following line:

```
add_subdirectory(apps/4dsound)
```

Setting the `QT_DIR` environment variable
---
We also need to tell the build scripts where to find our Qt installation. We do so by setting the value of an environment variable called QT_DIR to the location of the binary installation of Qt:

####  MacOS:
~~~
export QT_DIR=”~/Qt/5.12.0/clang_64”
~~~

#### Windows 10:
~~~
set QT_DIR=C:\Qt\5.12.0\win64
~~~

Make sure the location, version number and OS name are entered correctly when specifying the Qt path and verify that the specified path exists.

Running the package build
---
After making these changes invoke the build script for the platform package from the root directory of the nap repository:

#### On MacOS:
~~~
./package -nz -a
~~~

#### On Windows 10:
~~~
package -nz -a
~~~

This command invokes the cmake build system. -nz means that the platform package will not be zipped and -a means that application projects (in our case the 4DSOUND Engine) will be included.

Let the build run for a while. When the script is finished you should be left with a directory that looks somewhat like this:

`NAP-0.4.2-macOS-2022.04.13T13.17`

The directory name contains the Nap framework version, the OS name and the date and time of the build.

Building the 4DSOUND engine application
---
Last step is to build the 4DSOUND engine application from the platform release package:

Cd into the directory that was created in the previous step (make sure to edit the directory name to your own):

~~~
cd NAP-0.4.2-macOS-2022.04.13T13.17
cd projects
cd 4dsound
~~~

Then run the build:

#### On MacOS:

~~~
./package -nz
~~~


#### On Windows:

~~~
package -nz
~~~

You should be left with a directory that looks like this:

`4dsound-1.0.3-macOS-2022.04.19T16.11`

You can cd into this directory and run the application:

#### On MacOS:

~~~
./4dsound
~~~


#### On Windows:

~~~
4dsound
~~~
