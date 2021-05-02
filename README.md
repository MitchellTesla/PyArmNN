# PyArmNN-MT
NOTE: This repository is experimental and was built for Linux integration testing and development.
Please refer to the original repo for clone or copying.

PyArmNN is a python extension for [Arm NN SDK](https://developer.arm.com/ip-products/processors/machine-learning/arm-nn) and provides an interface similar to Arm NN C++ API.

This repository provides resources to create legacy Python packages for Arm NN 19.08, 19.11, and 20.02. Since 20.05 PyArmNN is integrated into Arm NN itself. It is also targetted for the [NXP i.MX8 series](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-8-processors:IMX8-SERIES), which runs [Yocto Linux](https://www.yoctoproject.org/) on aarch64, other platforms are not tested.

Before you proceed with building or installing, you will need to checkout and build a corresponding [Arm NN version](https://source.codeaurora.org/external/imx/armnn-imx/). You may also find the libraries and header files in `/usr/lib` and `/usr/include` on your Yocto image.

PyArmNN is built around public Arm NN headers. PyArmNN does not implement any computation kernels itself, all operations are
delegated to the Arm NN C++ library.

The [SWIG](http://www.swig.org/) project is used to generate the Arm NN python shadow classes and C wrapper.

# Setup development environment

To build Python packages or develop PyArmNN, you will need to set up a working environment. You do not need to do this to install a Python package, run examples, or unit tests. To set up your environment, make sure that:

1. You have Python 3.6+ installed system-side. The package is not compatible with older Python versions.
2. You have python3.6-dev installed system-side. This contains header files needed to build PyArmNN extension module.
3. In case you build Python from sources manually, make sure that the following libraries are installed and available in your system:
``python3.6-dev build-essential checkinstall libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev``
4. Install SWIG 4.x. Only 3.x version is typically available in Linux package managers, so you will have to build it and install it from sources. It can be downloaded from the [SWIG project website](http://www.swig.org/download.html) or [SWIG GitHub](https://github.com/swig/swig). To install it follow the guide on [SWIG GitHub](https://github.com/swig/swig/wiki/Getting-Started).

## Setup virtual environment

After setting up the development environment, it is recommended to create a Python virtual environment, so you do not pollute your working folder:
```bash
python3 -m venv env
source env/bin/activate
```

You may run into missing python modules such as *wheel*. Make sure to install those using pip:
```bash
pip install wheel
```

## Building Python packages

Python supports source and binary distribution packages.

The source package contains `setup.py` script that is executed on the user's machine during package installation, thus it is platform-independent.

When preparing the binary package (wheel), `setup.py` is executed on the build machine, the resulting package contains only the result
of the build (generated files and resources, test results, etc.) and so it is platform dependent.

There are 2 ways to build PyArmNN Python packages. Either using individual scripts or using CMake, which automates the process.

### CMake build

The recommended aproach is to build PyArmNN using CMake. First run the cmake command from the build directory and then run make:
```bash
mkdir build
cd build
cmake .. \
-DBUILD_PYTHON_SRC=<0 or 1> \
-DBUILD_PYTHON_WHL=<0 or 1> \ 
-DARMNN_LIB=<path_to_armnn_libs> \
-DARMNN_INCLUDE=<path_to_armnn_headers> \
-DSWIG_EXECUTABLE=<path_to_swig_executable>
make -j<no_of_threads>
```
`BUILD_PYTHON_SRC` can be set to 0 or 1 depending on if you want to build the python source package. 

`BUILD_PYTHON_WHL` can be set to 0 or 1 depending on if you want to build the python wheel (binary package).

`ARMNN_LIB` should point to the directory where Arm NN libraries can be found.

`ARMNN_INCLUDE` should point to the directory where Arm NN headers can be found.

`SWIG_EXECUTABLE` is optional. Run `swig -version` to see what version of swig is called. If it's 4.x, you are good to go, otherwise, you should set this variable to point to the swig executable (not the directory).

After the build finishes, you will find the python packages in `<build_dir>/python/pyarmnn/dist`.

### Build using individual scripts

PyArmNN can also be built using the provided python scripts only. The advantage of that is that you may use prebuilt Arm NN libraries, and generally have more control over the build.

First, navigate to the `<repo_dir>/python/pyarmnn` directory.
```bash
cd <repo_dir>/python/pyarmnn
```

#### 1. Set environment:
`ARMNN_INCLUDE` and `ARMNN_LIB` are mandatory and should point to Arm NN headers and libraries against which you will be generating the wrappers. `SWIG_EXECUTABLE` should only be set if you have multiple versions of SWIG installed or you used a custom location for your installation. You should also run the commands from the `<repo_dir>/python/pyarmnn` directory:
```bash
export SWIG_EXECUTABLE=<path_to_swig_exec>
export ARMNN_INCLUDE=<path_to_armnn_include>
export ARMNN_LIB=<path_to_armnn_libraries>
``` 

#### 2. Clean and build SWIG wrappers:

```bash
python setup.py clean --all
python swig_generate.py -v
python setup.py build_ext --inplace
```
This step will put all generated files under `src/pyarmnn/_generated` folder and can be used repeatedly to re-generate the wrappers.

#### 4. Build the source package

```bash
python setup.py sdist
```
As the result you will get `<build_dir>/python/pyarmnn/dist/pyarmnn-20.2.0.tar.gz` file. As you can see it is platform-independent.

#### 5. Build the binary package

```bash
python setup.py bdist_wheel
```
As the result you will get something like `dist/pyarmnn-20.2.0-cp37-cp37m-linux_aarch64.whl` file. As you can see it is platform-dependent.

# PyArmNN installation

PyArmNN can be distributed as a source package or a binary package (wheel).

Binary package is platform dependent, the name of the package will indicate the platform it was built for, e.g.:

* Linux x86 64bit machine: `pyarmnn-20.2.0-cp37-cp37m-linux_x86_64.whl`
* Linux Aarch 64 bit machine: `pyarmnn-20.2.0-cp37-cp37m-linux_aarch64.whl`

The source package is platform-independent but installation involves the compilation of Arm NN Python extension. You will need to have g++ compatible with C++ 14 standard and a python development library installed on the build machine.

Both of them, source and binary package, require the Arm NN library to be present on the target/build machine.

It is strongly suggested to work within a python virtual environment. The further steps assume that the virtual environment was created and activated before running PyArmNN installation commands.

PyArmNN also depends on the NumPy python library. It will be automatically downloaded and installed alongside PyArmNN. If your machine does not have access to Python pip repositories you might need to install NumPy in advance by following public instructions: https://scipy.org/install.html

## Installing from the wheel

Make sure that Arm NN binaries and Arm NN dependencies are installed and can be found in one of the system default library locations. You can check default locations by executing the following command:
```bash
gcc --print-search-dirs
```
Install PyArmNN from binary by pointing to the wheel file:
```bash
pip install dist/pyarmnn-20.2.0-cp37-cp37m-linux_aarch64.whl
```

## Installing from the source package

Alternatively, you can install it from the source package. This is the more reliable way but requires a little more effort on the users part.

While installing from sources, you have the freedom of choosing Arm NN libraries' location. Set environment variables `ARMNN_LIB` and `ARMNN_INCLUDE` to point to Arm NN libraries and headers.
If you want to use system default locations, just set `ARMNN_INCLUDE` to point to Arm NN headers.

```bash
export ARMNN_INCLUDE=<path_to_armnn_include>
export ARMNN_LIB=<path_to_armnn_libraries>
```

Install PyArmNN as follows:
```bash
pip install dist/pyarmnn-20.2.0.tar.gz
```

If PyArmNN installation script fails to find Arm NN libraries it will raise an error like this

`RuntimeError: ArmNN library was not found in ('/usr/lib/gcc/aarch64-linux-gnu/8/', <...> ,'/lib/', '/usr/lib/'). Please install ArmNN to one of the standard locations or set correct ARMNN_INCLUDE and ARMNN_LIB env variables.`

You can now verify that PyArmNN library is installed and check PyArmNN version using:
```bash
pip show pyarmnn
```
You can also verify it by running the following and getting output similar to below:
```bash
python -c "import pyarmnn as ann;print(ann.GetVersion())"
20200200
```

# PyArmNN API overview

### Getting started
The easiest way to begin using PyArmNN is by using Parsers. We will demonstrate how to use them below:

Create a parser object and load your model file.
```python
import pyarmnn as ann
import imageio

# ONNX, Caffe and TF parsers also exist.
parser = ann.ITfLiteParser()
network = parser.CreateNetworkFromBinaryFile('./model.tflite')
```

Get the input binding information by using the name of the input layer.
```python
input_binding_info = parser.GetNetworkInputBindingInfo(0, 'model/input')

# Create a runtime object that will perform inference.
options = ann.CreationOptions()
runtime = ann.IRuntime(options)
```
Choose preferred backends for execution and optimize the network.
```python
# Backend choices earlier in the list have higher preference.
preferredBackends = [ann.BackendId('CpuAcc'), ann.BackendId('CpuRef')]
opt_network, messages = ann.Optimize(network, preferredBackends, runtime.GetDeviceSpec(), ann.OptimizerOptions())

# Load the optimized network into the runtime.
net_id, _ = runtime.LoadNetwork(opt_network)
```
Make workload tensors using input and output binding information.
```python
# Load an image and create an inputTensor for inference.
img = imageio.imread('./image.png')
input_tensors = ann.make_input_tensors([input_binding_info], [img])

# Get output binding information for an output layer by using the layer name.
output_binding_info = parser.GetNetworkOutputBindingInfo(0, 'model/output')
output_tensors = ann.make_output_tensors([outputs_binding_info])
```

Perform inference and get the results back into a NumPy array.
```python
runtime.EnqueueWorkload(0, input_tensors, output_tensors)

results = ann.workload_tensors_to_ndarray(output_tensors)
print(results)
```

### Examples

To further explore PyArmNN API there are several examples provided in the examples folder running classification on an image. To run them first install the dependencies:
```bash
cd python/pyarmnn
pip install -r examples/requirements.txt
```

Afterward simply execute the example scripts, e.g.:
```bash
python tflite_mobilenetv1_quantized.py
```
All resources are downloaded during execution, so if you do not have access to the internet, you may need to download these manually. `example_utils.py` contains code shared between the examples.

# Tox for automation

To make things easier *tox* is available for automating individual tasks or running multiple commands at once such as generating wrappers, running unit tests using multiple python versions, or generating documentation. To run it use:

```bash
tox <task_name>
```

See `tox.ini` for the list of tasks. You may also modify it for your own purposes. To dive deeper into tox read through https://tox.readthedocs.io/en/latest/.

Tox is intended for easier automation, so you may have to modify the script slightly to work with your environment. We do not guarantee, that it works out of the box.

# Running unit-tests

Download resources required to run unit tests by executing the script from the `python/pyarmnn` directory. You will also need to install the required python modules.

```bash
cd python/pyarmnn
pip install -r test/requirements.txt
python ./scripts/download_test_resources.py
```

The script will download an archive from the Linaro server and extract it. A folder `test/testdata` will be created. Execute `pytest` from the project root dir:
```bash
python -m pytest test/ -v
```
or run tox which will do both:
```bash
tox
```

# Prebuilt wheels

Prebuilt wheel packages for aarch64 used in Yocto Linux images can be found in the `whl` directory. There are many architectures, versions of Python, etc., so if the package is not there, you will need to build it yourself.
