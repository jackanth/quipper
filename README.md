# quipper

quipper facilitates the reading of perf.data files generated by the Linux sampling profiler perf (https://github.com/torvalds/linux/tree/master/tools/perf). 

This repository takes the quipper subdirectory of google/perf_data_converter and builds it as a standalone library. The code is kept up-to-date with google/perf_data_converter.

For more information, and to submit issues, visit the Google repo at https://github.com/google/perf_data_converter/

# Example build on clean Ubuntu 18.04
* Install dependencies plus git, cmake and a compiler.
```
sudo apt update
sudo apt install libssl-dev libelf-dev git cmake clang 
```
* Install protobuf >v3.0.0 if not already installed
```
sudo apt install libprotobuf-dev protobuf-compiler
```

* Build quipper
```
git clone https://github.com/jackanth/quipper.git
cd quipper
mkdir build
cd build
cmake ..
make -j4
sudo make install
```
