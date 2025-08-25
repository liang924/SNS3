# SNS3 Installation

> Reference :
> - https://github.com/sns3/sns3-satellite?tab=readme-ov-file#bake

### Create & Enter Project Folder
```
mkdir ~/sns3
cd ~/sns3
```
### Install dependency
```
sudo apt update
sudo apt install -y git automake cmake qtbase5-dev python3-dev python3
```

> ## Setting up sns3 Environment (with Virtualenv)
> 1. Install venv tool
> ```
> sudo apt install python3-venv
> ```
> 2. Create a virtual environment (inside the sns3 folder)
> ```
> python3 -m venv venv
> ```
> 3. Activate the virtual environment
> ```
> source venv/bin/activate
> ```
> **Output**
> <img width="1219" height="62" alt="image" src="https://github.com/user-attachments/assets/2565b9aa-5ad1-44e9-a44f-9f1a9d103fd8" />
> 4. Install dependencies inside the virtual environment
> ```
> pip install --upgrade pip setuptools cppyy pygraphviz
> pip install --upgrade pip setuptools wheel
> pip install distro
> pip install requests 
> ```
> ### How to use sns3 later
> Every time you want to work on sns3, navigate to the project folder and activate the virtual environment again:
> ```
> cd ~/sns3
> source venv/bin/activate
> ```
### Installation by bake
```
cd ~
mkdir workspace
cd workspace
git clone https://gitlab.com/nsnam/bake.git
```
**Output**

<img width="500" height="210" alt="image" src="https://github.com/user-attachments/assets/db31970e-9ce0-465e-8c06-f5e22eb26126" />

### Set environment parameters:
```
export BAKE_HOME=`pwd`
export PATH=$PATH:$BAKE_HOME/build/bin
export PYTHONPATH=$BAKE_HOME/build/lib
export LD_LIBRARY_PATH=$BAKE_HOME/build/lib
```
```
cd bake
./bake.py configure -e ns-allinone-3.44
./bake.py check
```
**Output**

<img width="561" height="275" alt="image" src="https://github.com/user-attachments/assets/dab39136-cd2f-4237-891c-59435b6231ed" />

### Deploy ns3 by bake
```
./bake.py download
```
**Output**

<img width="543" height="220" alt="image" src="https://github.com/user-attachments/assets/d51aff42-c342-4ed1-8a86-e3ae981992ae" />

### Configure ns3
```
cd source/ns-3.44
./ns3 clean
./ns3 configure --build-profile=debug --enable-examples --enable-tests
./ns3 configure --build-profile=debug --enable-examples --enable-tests
```
**Output**

<img width="1789" height="249" alt="image" src="https://github.com/user-attachments/assets/557f6473-039a-4647-996e-0cd7d35ff42c" />
<img width="1791" height="218" alt="image" src="https://github.com/user-attachments/assets/8d185358-cd9b-4176-a3c2-f4674550b757" />

### Get satellite/ traffic/ma modules
```
cd contrib
git clone https://github.com/sns3/sns3-satellite.git satellite
git clone https://github.com/sns3/traffic.git traffic
git clone https://github.com/sns3/stats.git magister-stats
```
> Note: When retrieving the satellite, traffic, and magister-stats modules, you should place them under the ns-3.44/contrib/ folder.
> You can do this by cloning them directly in this folder, copying them here after downloading, or using symbolic links.
> Make sure that all repositories are using compatible versions. The best way to ensure this is to use the same tag on all repositories.
> For the latest release, please use: ns-3.44.
>
> -  On NS-3 repository:
>   ```
>   # Go into the satellite module and switch to 3.44
>   cd ~/workspace/bake/source/ns-3.44/contrib/satellite
>   git checkout 3.44
>   # Go back to contrib and enter the traffic module
>   cd ../traffic
>   git checkout 3.44
>   # Go back to contrib again and enter the magister-stats module
>   cd ../magister-stats
>   git checkout 3.44

### Configure CMake and ask it to build NS-3 
It will automatically build all modules found in contrib:
```
cd ns-3.44
./ns3 clean
./ns3 configure --build-profile=optimized --enable-examples --enable-tests
./ns3 build
```
**Output**

<img width="1543" height="156" alt="image" src="https://github.com/user-attachments/assets/20205ab4-762c-49c7-abbc-2a30c867cf91" />

You can also check CMake options to customize it at will:
```
$ ./ns3 --help
```

### Post-Compilation
Once you compiled SNS-3 successfully, you will need an extra step before being able to run any simulation: download the data defining the reference scenario of the simulation.
These data are available as a separate repository and bundled as a submodule in SNS-3. You can download them afterwards in the `satellite` repository using:
```
cd source/ns-3.43/contrib/satellite
git submodule update --init --recursive
```
