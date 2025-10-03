# Lab 1 - Installing Snort 3
This lab is going to focus on installing Snort 3 and  it's dependancies. This is being done on a Kali Linux VM

# 1 - Update, Upgrade, Install
1. First let's update and upgrade our system
   ```
   sudo apt update && sudo apt -y upgrade
   ```
2. Install the latest version of Snort
   ```
   sudo apt -y install snort
   ```
3. Check the version using `snort -V`. If successfull, you should see something like:
   <img width="656" height="241" alt="image" src="https://github.com/user-attachments/assets/ce97c3df-e2ff-413c-bbb9-a7761b6aedd4" />
# 2 - Installing Snort 3 Dependancies
As you can see from the screenshot above, the Snort package from Debian comes with a lot of of the required packages already installed:
- DAQ
- LuaJIT
- OpenSSL
- libpcap
- PCRE
- ZLIB
- LZMA
## 2.1 - Hyperscan
One thing missing is [Hyperscan](http://intel.github.io/hyperscan/dev-reference/intro.html), which is a high-performance, open-source regular expression matching engine originally developed by Intel. You can use Snort 3 without Hyperscan, but you'll be hit with some pretty significant tradeoffs:
- Slower performance with alrge rulesets
- Potential scaling limitations with high-throughput environments
- Snort 3 will not be as "future proof" since newer rulesets may leverage this

First, clone the Hyperscan git repo to your preferred location (for me I keep all my git repos in ~/git_packages)
```
git clone https://github.com/intel/hyperscan.git
```
## 2.2 - Hyperscan Dependancies
Now we begin dependancy hell!

Hyperscan itself has a few dependancies that we need to install. Just get the latest versions for each and you should be fine.

<img width="669" height="257" alt="image" src="https://github.com/user-attachments/assets/c2e76120-95c3-4ed1-a4a1-cc1484e06d36" />

1. Install CMake
   ```
   sudo apt install -y cmake
   ```
   Check if CMake installed correcly
   ```
   cmake --version
   ```
2. Python should already be installed on your Kali Linux machine since it's included in the package. You can confirm by running the command below to make sure you at least have the version required
   ```
   python --version
   ```
   If you don't have Python installed:
   ```
   sudo apt -y install python3 python3-pip
   ```
### 2.3.1 - Ragel
Installing [Ragel](http://www.colm.net/open-source/ragel/) requires a few extra steps. Ragel is a state machine compiler
1. Download the latest tar ball form the official website (I installed it in my ~/Downloads directory using wget from the temrinal)
   ```
   cd ~/Downloads
   wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz
   ```
2. Extract the tar ball. Choose whatever target location you like I just kept it in the Downloads directory
   ```
   tar -xvzf /path/to/ragel.tar.gz 
   ```
3. Move into the extracted Ragel directory and run the configuration. This will configure your system for Ragel and generate the makefile
   ```
   cd /path/to/extracted/ragel/directory
   ./configure
   ```
4. Compile the source code into binaries
   ```
   make
   ```
5. Now actually install Ragel
   ```
   sudo make install
   ```
6. If all went well, run `ragel --version` and you should see something like:
   ```
   Ragel State Machine Compiler version 6.10 March 2017
   Copyright (c) 2001-2009 by Adrian Thurston
   ```
#### 2.3.1.1 - Ragel Dependancies (COLM)
Ragel itself has a dependancy which is COLM (COmputer Language Machinery). [COLM](https://github.com/adrian-thurston/colm) is a programming language for language processors...whatever that means, but was actually developed my the same person who created Ragel.

1. Clone the COLM repo to your desired location (again, I did ~/git_packages)
   ```
   git clone https://github.com/adrian-thurston/colm.git
   ```
2. Move into the COLM directory and this time we need to do some bootstrapping and generate the configuration script
   ```
   cd /path/to/colm
   ./autogen.sh
   ```
3. Now we can run the configuration and generate the makefile
   ```
   ./configure
   ```
4. Compile and create binaries
   ```
   make
   ```
5. Install colm
   ```
   sudo make install
   ```
All done!
### 2.3.2 - Boost
This is the final dependancy to install. [Boost](https://www.boost.org/) is a collection of C++ libraries. The GitHub repo for Boost can be found [here](https://github.com/boostorg/boost?tab=readme-ov-file)

Install Ragel
```
wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz
```

Extract Ragel tar
```
tar -xvzf /path/to/ragel.tar.gz
```

Configure
```
./configure
```

Make
```
make
```

Install
```
sudo make install
```

Check Ragel installation
```
ragel --version
```

Download Boost tar
```
wget https://archives.boost.io/release/1.89.0/source/boost_1_89_0.tar.gz
```


