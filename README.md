# Faster R-CNN C++ Inference

The above code is an interface inference in c++ against a [Faster R-CNN](https://github.com/rbgirshick/py-faster-rcnn) trained network.

From the code developed by Shaoqing Ren, Kaiming He, Ross Girshick, Jian Sun (Microsoft Research) for the project [py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn), some modules and files from the `py-faster-rcnn/lib` and `py-faster-rcnn/data/scripts/` folders are used directly:
 - **RPN module** (`py-faster-rcnn/lib/rpn/`): Needed to deploy the *region proposal network*.
 - **NMS module** (`py-faster-rcnn/lib/nms/`): Needed to apply *non-maximum suppression* step.
 - **Fast_rcnn module** (`py-faster-rcnn/lib/fast_rcnn/`): Contains auxiliary functions.
 - **`lib/Makefile` and `py-faster-rcnn/lib/setup.py`**: To compile NMS CUDA and Cython libraries.
 - **`py-faster-rcnn/data/scripts/fetch_faster_rcnn_models.sh`**: To download pre-computed Faster R-CNN detectors.

This code is added as a submodule to the present project for convenience. It also uses their branch of the framework Caffe ([caffe-fast-rcnn](https://github.com/rbgirshick/caffe-fast-rcnn/tree/0dcd397b29507b8314e252e850518c5695efbb83)).

`FASTER_RCNN` C++ class implemented is a modification/correction of the project [FasterRCNN-Encapsulation-Cplusplus](https://github.com/YihangLou/FasterRCNN-Encapsulation-Cplusplus) fixing some bugs and extending its functionality to more than two categories.

### Steps to use the code

First of all, clone this project with **`--recursive`** flag:
```Shell
git clone --recursive https://github.com/HermannHesse/faster_rcnn_cplusplus.git
```

#### Requeriments inherited from py-faster-rcnn

1. Requirements for `Caffe` and `pycaffe` (see: [Caffe installation instructions](http://caffe.berkeleyvision.org/installation.html))

    **Note:** Caffe *must* be built with support for Python layers!
    ```make
    # In your Makefile.config, make sure to have this line uncommented
    WITH_PYTHON_LAYER := 1
    # Unrelatedly, it's also recommended that you use CUDNN
    USE_CUDNN := 1
      ```
    You can download Ross Girshick [Makefile.config](http://www.cs.berkeley.edu/~rbg/fast-rcnn-data/Makefile.config) for reference.
  
2. Python packages you might not have: `cython`, `python-opencv`, `easydict`

3. Build Caffe and pycaffe
    ```Shell
    cd $ROOT_DIR/py-faster-rcnn/caffe-fast-rcnn/
    # Now follow the Caffe installation instructions here:
    #   http://caffe.berkeleyvision.org/installation.html

    # If you're experienced with Caffe and have all of the requirements installed
    # and your Makefile.config in place, then simply do:
    make -j8 && make pycaffe
    ```
    
4. Build the Cython modules
    ```Shell
    cd $ROOT_DIR/py-faster-rcnn/lib/
    make
    ```

5. Download pre-computed Faster R-CNN detectors
    ```Shell
    cd $ROOT_DIR/py-faster-rcnn/
    ./data/scripts/fetch_faster_rcnn_models.sh
    ```
    
#### Own requeriments

1. Set `$ROOT_DIR/py-faster-rcnn/caffe-fast-rcnn/python/` and `$ROOT_DIR/py-faster-rcnn/lib/` into the enviroment variable `PYTHONPATH`
    ```Shell
    export PYTHONPATH=$ROOT_DIR/py-faster-rcnn/caffe-fast-rcnn/python/:$ROOT_DIR/py-faster-rcnn/lib/:${PYTHONPATH}
    ```

2. Compile the project with CMake
    ```Shell
    mkdir $ROOT_DIR/build/
    cd $ROOT_DIR/build/
    cmake ..
    make
    mv faster_rcnn_cplusplus ..
    ```

3. Run the demo
    ```Shell
    cd $ROOT_DIR
    ./faster_rcnn_cplusplus
    ```
    
### Popular issues

#### Issue 1

```Shell
"Unknown layer type: Python"
```
Caffe has been probably compiled without PYTHON_LAYER support. Uncomment the following line 
```make
WITH_PYTHON_LAYER := 1
```
in `$ROOT_DIR/py-faster-rcnn/caffe-fast-rcnn/Makefile.config` file and recompile it.
```Shell
cd $ROOT_DIR/py-faster-rcnn/caffe-fast-rcnn/
make clean
make -j8 && make pycaffe
```

#### Issue 2

```Shell
fatal error: caffe/proto/caffe.pb.h: No such file or directory
#include "caffe/proto/caffe.pb.h"
                                  ^
compilation terminated.
```
`caffe.pb.h` is a header file generated by Google Protocol Buffer and it is missing for some reason. Let's create it.
```Shell
cd $ROOT_DIR/py-faster-rcnn/caffe-fast-rcnn/
protoc src/caffe/proto/caffe.proto --cpp_out=.
mkdir include/caffe/proto
mv src/caffe/proto/caffe.pb.h include/caffe/proto
```
Reference full discussion from [here](https://github.com/NVIDIA/DIGITS/issues/105).

#### Issue 3

When the net is loading Caffe can't find rpn python layer:
```Shell
ImportError: No module named rpn.proposal_layer
```
Add `lib/` directory to `PYTHONPATH`:
```Shell
export PYTHONPATH=$ROOT_DIR/py-faster-rcnn/lib/:${PYTHONPATH} 
```

#### Issue 4

Python can't find gpu_nms library:
```Shell
from nms.gpu_nms import gpu_nms
ImportError: No module named gpu_nms
```
Must compile libraries `gpu_nms.so` and `cpu_nms.so`done by running the fourth step:
```Shell
cd $ROOT_DIR/py-faster-rcnn/lib/
make
```

#### Issue 5

```Shell
/usr/bin/ld: cannot find -lopencv_dep_cudart 
collect2: error: ld returned 1 exit status
```
There is a problem with the `CMakeCache.txt` and the enviroment variable `CUDA_USE_STATIC_CUDA_RUNTIME`.
To solve this set it `OFF` in `CMakeList.txt`: 
```Shell
# Add the following line to CMakeList.txt and recompile
set(CUDA_USE_STATIC_CUDA_RUNTIME "OFF")
```
or generate CMake files with:
```Shell
cd $ROOT_DIR/build/
cmake .. -D CUDA_USE_STATIC_CUDA_RUNTIME=OFF
```
Reference full discussion from [here](https://github.com/opencv/opencv/issues/6542).
