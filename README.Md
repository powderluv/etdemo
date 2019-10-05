Bazel sucks your life if you try to build on natively on aarch64 so here is an Coral EdgeTPU CPP demo based only Cmake


```
# Install libraries needed
sudo apt install -y curl wget cmake ninja-build
sudo apt install -y libc6-dev libc++-dev libc++abi-dev
sudo apt install -y libusb-1.0-0 


# Clone and download code
cd ~/
git clone https://github.com/powderluv/etdemo/
mkdir build && cd build
cmake .. -GNinja
./etdemo

#if you are on newer distros that dont have the deps like GEntoo 64bti on RP4 
LD_LIBRARY_PATH="../prebuilts/aarch64" ./etdemo
```

Based off work from: https://github.com/iwatake2222/CNN_NumberDetector
