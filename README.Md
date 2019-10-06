Bazel sucks your life if you try to build on natively on aarch64 so here is an Coral EdgeTPU CPP demo based only Cmake


```
# Install libraries needed
sudo apt install -y curl wget cmake ninja-build libusb-1.0-0 

# Clone and download code
cd ~/
git clone https://github.com/powderluv/etdemo/
mkdir build && cd build
cmake .. -GNinja
./etdemo

LD_LIBRARY_PATH="." ./etdemo
```

Based off work from: https://github.com/iwatake2222/CNN_NumberDetector
