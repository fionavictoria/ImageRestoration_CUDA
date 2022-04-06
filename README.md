# Accelerating Image Restoration using CUDA
The image restoration algorithm involves optimizing the mathematical computations applied to the degraded image to result in a better resolution image. The process is done heuristically without using in-built libraries for most of the code to eliminate library dependencies. The algorithm is thus written from scratch to ensure the acceptable use of native language capabilities. In the second stage, the algorithm is optimized for each loop to enhance performance. The performance analysis report for the algorithm's runtime is shown in the results. The promising results indicate a gain in acceleration achieved in the performance with a change in hardware and algorithm design.

## Hardware setup
| CPU | Intel(R) Core(TM) i9-9900K |
| ------ | ------ |
| Memory | 128 GB |
| Cores | 8 (16 Threads) |
| Clock Speed | Base = 3.60 GHz, Max = 5.00 GHz |
| Instruction per clock cycle | FP32 = 32, FP64 = 16 |
| Caches | 16 MB Intel® Smart Cache - L1 - 256 KB (code) / 256 KB (data), L2 - 2048KB, L3 - 16384KB |

| GPU | NVIDIA GeForce RTX 3070 |
| ------ | ------ |
| Architecture and Compute Capability | Ampere, 8.6 |
| Cores | 5888 |
| Clock Speed | 1500 MHz |
| Memory | 8GB |
| Bandwidth | 448.0 GB/s |
| Theoretical Performance | FP 32 - 20.31 TFLOPS, FP 64 - 317.4 GFLOPS |

## Tools
- CUDA Version - V11.2
- OpenCV library with CUDA compatibility - V3.2.0
- Visual Studio Code - V1.65.2
- Nsight profiler

## Instructions to compile and execute
- Store the input high quality images (.png) in a particular directory, say 'images'. Add the complete file path as input to the imread function of all the .cpp and .cu implementations
- Modify the values of R and SNR for the point spread function to enhance quality of image restoration. 

#### CPU Mode 
##### OpenCV
- C++ OpenCV Implementation of Image restoration

``` 
g++ Opencv_implementation.cpp -o program -I/usr/local/include/opencv4 -lopencv_core -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc $(pkg-config opencv4 --libs)
``` 

##### Sequential
- C++ Sequential Implementation of Image restoration (Manual - without Opencv's built-in functions)

``` 
g++ Sequential_implementation.cpp -o program -I/usr/local/include/opencv4 -lopencv_core -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc $(pkg-config opencv4 --libs)
```

##### Auto-Tune
- C++ Auto-Tune OpenCV Implementation of Image restoration
- Helps determine the best values of R and snr for a given image when performing PSF function based image restoration 

``` 
g++ findbestparams.cpp -o program -I/usr/local/include/opencv4 -lopencv_core -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc $(pkg-config opencv4 --libs)
```

#### GPU Mode
``` 
nvcc -std=c++11 Cuda_implementation -o program -I/usr/local/include/opencv4 -lopencv_core -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc $(pkg-config opencv4 --libs)
``` 
#### Execution 
- Without profilers
```
./program
```
- With profilers
```
nv-nsight-cu-cli ./program
nsys profile --stats=true --force-overwrite true --show-output true ./program
```
##### Note
- The above implemented image restoration algorithm works well on the square sized and even sized images
- Maximum recommemded image size - 256 * 256 pixels

## More information
On CUDA kernel usage, image restoration results and performance analysis -  [ImageRestoration_CUDA.pdf](https://github.com/fionavictoria/ImageRestoration_CUDA/blob/main/ImageRestoration_CUDA.pdf)
   
> The proposed design and algorithm can be extended to real-time image processing applications incorporated on edge devices. In addition, as the restoration acceleration is achieved, there is a scope for applying the same to video-based platforms where the video streams are dealt with frames

-----------------------------


#### Help with OpenCV – CUDA setup
```
mkdir "$USER"
cd "$USER" && mkdir opencv_build && cd opencv_build
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
source `which virtualenvwrapper.sh`
mkvirtualenv opencv_cuda -p python3.9
cd opencv && mkdir build && cd build
pip install numpy

cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D INSTALL_PYTHON_EXAMPLES=ON -D INSTALL_C_EXAMPLES=OFF -D OPENCV_ENABLE_NONFREE=ON -D WITH_CUDA=ON -D WITH_CUDNN=ON -D OPENCV_DNN_CUDA=ON -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D CUDA_ARCH_BIN=8.6 -D WITH_CUBLAS=1 -D OPENCV_EXTRA_MODULES_PATH="$USER"/opencv_build/opencv_contrib/modules -D HAVE_opencv_python3=ON -D PYTHON_EXECUTABLE=~/.virtualenvs/opencv_cuda/bin/python -D BUILD_EXAMPLES=ON ..

make -j16
make install
cd ~/.virtualenvs/opencv_cuda/lib/python3.9/site-packages/
ln -s /usr/local/lib/python3.9/site-packages/cv2/python-3.9/cv2.cpython-39-x86_64-linux-gnu.so
cd "$USER"
```

To test OpenCV – CUDA installation.
```
python3
>>import cv2
>>count = cv2.cuda.getCudaEnabledDeviceCount()
>>print(count)
>>1
>>print(cv2.getBuildInformation())
```
*Should get a 1 if installation was successful. CTRL+D to exit python3.
*CUDA settings should be enabled in cv2 build.

