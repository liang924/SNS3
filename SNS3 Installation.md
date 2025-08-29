# SNS3 Installation

> Reference :
> - https://github.com/sns3/sns3-satellite?tab=readme-ov-file#bake

### Install dependency
```
sudo apt update
sudo apt install -y git automake cmake qtbase5-dev python3-dev python3
```

### Installation by bake
```
cd ~
mkdir workspace
cd workspace
git clone https://gitlab.com/nsnam/bake.git
```
**Output**
```
Cloning into 'bake'...
remote: Enumerating objects: 2332, done.
remote: Counting objects: 100% (368/368), done.
remote: Compressing objects: 100% (97/97), done.
remote: Total 2332 (delta 289), reused 341 (delta 271), pack-reused 1964 (from 1)
Receiving objects: 100% (2332/2332), 2.78 MiB | 4.85 MiB/s, done.
Resolving deltas: 100% (1569/1569), done.
```
### Set environment parameters:
```
export BAKE_HOME=`pwd`
export PATH=$PATH:$BAKE_HOME/build/bin
export PYTHONPATH=$BAKE_HOME/build/lib
export LD_LIBRARY_PATH=$BAKE_HOME/build/lib
```
```
cd bake
./bake.py configure -e ns-allinone-3.43
./bake.py check
```
**Output**
```
 > Python3 - OK
 > GNU C++ compiler - OK
 > Git - OK
 > Tar tool - OK
 > Unzip tool - OK
 > Make - OK
 > CMake - OK
 > patch tool - OK

 > Path searched for tools: /home/kitty/.vscode-server/cli/servers/Stable-6f17636121051a53c88d3e605c491d22af2ba755/server/bin/remote-cli /home/kitty/.local/bin /usr/local/sbin /usr/local/bin /usr/sbin /usr/bin /sbin /bin /usr/games /usr/local/games /snap/bin bin
```

### Deploy ns3 by bake
```
./bake.py download
```
**Output**
```
 >> Searching for system dependency qt - OK
 >> Searching for system dependency g++ - OK
 >> Searching for system dependency cppyy - OK
 >> Searching for system dependency gi-cairo - OK
 >> Searching for system dependency gir-bindings - OK
 >> Searching for system dependency pygobject - OK
 >> Searching for system dependency pygraphviz - OK
 >> Searching for system dependency python3-dev - OK
 >> Searching for system dependency libxml2-dev - OK
 >> Downloading click-ns-3.37 - OK
 >> Searching for system dependency automake - OK
 >> Searching for system dependency cmake - OK
 >> Searching for system dependency mercurial - OK
 >> Downloading netanim-3.109 - OK
 >> Downloading BRITE - OK
 >> Downloading openflow-dev - OK
 >> Downloading ns-3.43 (target directory:ns-3.43) - OK
```

### Configure ns3
```
cd source/ns-3.43
./ns3 clean
./ns3 configure --build-profile=debug --enable-examples --enable-tests
```
**Output**
```
-- Configuring done
-- Generating done
-- Build files have been written to: /home/kitty/workspace/bake/source/ns-3.43/cmake-cache
Finished executing the following commands:
mkdir cmake-cache
/usr/bin/cmake -S /home/kitty/workspace/bake/source/ns-3.43 -B /home/kitty/workspace/bake/source/ns-3.43/cmake-cache -DCMAKE_BUILD_TYPE=debug -DNS3_ASSERT=ON -DNS3_LOG=ON -DNS3_WARNINGS_AS_ERRORS=ON -DNS3_NATIVE_OPTIMIZATIONS=OFF -DNS3_EXAMPLES=ON -DNS3_TESTS=ON -G Unix Makefiles --warn-uninitialized
```
### Get satellite/ traffic/ma modules
```
cd contrib
git clone https://github.com/sns3/sns3-satellite.git satellite
git clone https://github.com/sns3/traffic.git traffic
git clone https://github.com/sns3/stats.git magister-stats
```
> Note: When retrieving the satellite, traffic, and magister-stats modules, you should place them under the ns-3.43/contrib/ folder.
> You can do this by cloning them directly in this folder, copying them here after downloading, or using symbolic links.
> Make sure that all repositories are using compatible versions. The best way to ensure this is to use the same tag on all repositories.
>
> -  On NS-3 repository:
>   ```
>   # Go into the satellite module and switch to 3.43
>   cd ~/workspace/bake/source/ns-3.43/contrib/satellite
>   git checkout 3.43
>   # Go back to contrib and enter the traffic module
>   cd ../traffic
>   git checkout 3.43
>   # Go back to contrib again and enter the magister-stats module
>   cd ../magister-stats
>   git checkout 3.43

### Configure CMake and ask it to build NS-3 
It will automatically build all modules found in contrib:
```
cd ~/workspace/bake/source/ns-3.43
./ns3 clean
./ns3 configure --build-profile=optimized --enable-examples --enable-tests
./ns3 build
```
**Output**
```
Finished executing the following commands:
/usr/bin/cmake --build /home/kitty/workspace/bake/source/ns-3.43/cmake-cache -j 3
```

> You can also check CMake options to customize it at will:
> ```
> ./ns3 --help
> ```
> **Output**
> ```
> ns-3 wrapper for the CMake build system
> usage: ns3 [-h] [--dry-run] [-j JOBS] [--quiet] [-v] {help,build,configure,clean,distclean,install,uninstall,run,shell,docs,show} ...
>   or: ns3 help [-h]
>   or: ns3 build [-h] [--dry-run] [-j BUILD_JOBS] [--quiet] [-v] [target ...]
>   or: ns3 configure [-h] [-d {debug,default,release,optimized,minsizerel}] [-G G] [--cxx-standard CXX_STANDARD] [--enable-asserts] [--disable-asserts] [--enable-des-metrics] [--disable-des-metrics]
>                     [--enable-build-version] [--disable-build-version] [--enable-clang-tidy] [--disable-clang-tidy] [--enable-dpdk] [--disable-dpdk] [--enable-eigen] [--disable-eigen]
>                     [--enable-examples] [--disable-examples] [--enable-gcov] [--disable-gcov] [--enable-gsl] [--disable-gsl] [--enable-gtk] [--disable-gtk] [--enable-logs] [--disable-logs]
>                     [--enable-monolib] [--disable-monolib] [--enable-mpi] [--disable-mpi] [--enable-ninja-tracing] [--disable-ninja-tracing] [--enable-precompiled-headers]
>                     [--disable-precompiled-headers] [--enable-python-bindings] [--disable-python-bindings] [--enable-tests] [--disable-tests] [--enable-sanitizers] [--disable-sanitizers]
>                     [--enable-static] [--disable-static] [--enable-sudo] [--disable-sudo] [--enable-verbose] [--disable-verbose] [--enable-warnings] [--disable-warnings] [--enable-werror]
>                     [--disable-werror] [--enable-modules ENABLE_MODULES] [--disable-modules DISABLE_MODULES] [--filter-module-examples-and-tests FILTER_MODULE_EXAMPLES_AND_TESTS] [--lcov-report]
>                     [--lcov-zerocounters] [--out OUTPUT_DIRECTORY] [--with-brite WITH_BRITE] [--with-click WITH_CLICK] [--with-openflow WITH_OPENFLOW] [--force-refresh] [--prefix PREFIX]
>                     [--trace-performance] [--dry-run] [--quiet] [-v]
>   or: ns3 clean [-h] [--dry-run]
>   or: ns3 distclean [-h] [--dry-run]
>   or: ns3 install [-h]
>   or: ns3 uninstall [-h]
>   or: ns3 run [-h] [--no-build] [--command-template COMMAND_TEMPLATE] [--cwd CWD] [--gdb] [--lldb] [-g] [--memray] [--heaptrack] [--perf] [--vis] [--enable-sudo] [--dry-run] [-j RUN_JOBS] [--quiet]
>               [-v]
>               [target]
>   or: ns3 shell [-h]
>   or: ns3 docs [-h] [--dry-run] [-v] {contributing,installation,manual,models,tutorial,sphinx,doxygen-no-build,doxygen,all}
>   or: ns3 show [-h] [--dry-run] [--quiet] [{profile,version,config,targets,all}]

>  positional arguments:
>  {help,build,configure,clean,distclean,install,uninstall,run,shell,docs,show}
>    help                Print a summary of available commands
>    build               Accepts a list of targets to build, or builds the entire project if no target is given
>    configure           Try "./ns3 configure --help" for more configuration options
>   clean               Removes files created by ns3
>    distclean           Removes files created by ns3, tests and documentation
>    install             Install ns-3
>    uninstall           Uninstall ns-3
>    run                 Try "./ns3 run --help" for more runtime options
>    shell               Try "./ns3 shell --help" for more shell options
>    docs                Try "./ns3 docs --help" for more documentation options
>    show                Try "./ns3 show --help" for more show options
>
> options:
>  -h, --help            Print a summary of available commands
>  --dry-run             Do not execute the commands.
>  -j JOBS, --jobs JOBS  Set number of parallel jobs.
>  --quiet               Don't print task lines, i.e. messages saying which tasks are being executed.
>  -v, --verbose         Print which commands were executed

### Post-Compilation
Once you compiled SNS-3 successfully, you will need an extra step before being able to run any simulation: download the data defining the reference scenario of the simulation.
These data are available as a separate repository and bundled as a submodule in SNS-3. You can download them afterwards in the `satellite` repository using:
```
cd contrib/satellite
git submodule update --init --recursive
```
**Output**
```
Submodule 'data' (https://github.com/sns3/sns3-data.git) registered for path 'data'
Cloning into '/home/kitty/workspace/bake/source/ns-3.43/contrib/satellite/data'...
Submodule path 'data': checked out 'da089030e20e7857a00d72c764aee40306dc0ee1'
```

### Testing SNS-3

cd ~/workspace/bake/source/ns-3.43
./test.py --no-build

**Output**
```
863 of 863 tests passed (863 passed, 0 skipped, 0 failed, 0 crashed, 0 valgrind errors)
```

### Running SNS-3
A scenario can be launched as follows:
```
./ns3 run <ns3-program>
```
If command line arguments are needed, the command becomes:
```
./ns3 run <ns3-program> -- --arg1=value1 --arg2=value2
```
To list all the available command line arguments of a scenario, run:
```
./ns3 run <ns3-program> -- --PrintHelp
```
If debug mode is enabled, gdb can be used:
```
./ns3 run --gdb <ns3-program>
```

### SNS-3 is delivered with several examples. Each one allows to demonstrate one or several functionalities of SNS-3. Statistics are generated in the data/sims/<ns3-program> folder.

The main examples are:

- `sat-cbr-example.cc`: Simple example with CBR traffic
  
- `sat-regeneration-example.cc`: Example to test several regeneration modes on satellite: transparent, physical, link or network. By default, all simulations use one transparent satellite
  
- `sat-constellation-example.cc`: Example with LEO and GEO satellite constellations. ISLs are used to route packets between satellites, with static routing
  
- `sat-vhts-example.cc`: Create a VHTS scenario (high throughputs and high link capacities)
  
- `sat-iot-example.cc`: Create an IoT scenario (low throughputs and low link capacities)

- `sat-logon-example.cc`: Use the logon functionality to log the UTs on the NCC. Traffic on return channels is not send before UT is logged

- `sat-ncr-example.cc`: Use NCR synchronization between UTs and GWs. UT clock is generally cheap, and need to be resynchronized periodically by the NCC to correctly schedule sending of frames on return channel
  
- `sat-lora-example.cc`: Create a scenario with Lora configuration, and on transparent satellite. Lora is a LPWAN protocol developed for IoT
  
- `sat-lora-regenerative-example.cc`: Create a scenario with Lora configuration and regenerative satellites
