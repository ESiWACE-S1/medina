# MECCA - KPP Fortran to CUDA source-to-source pre-processor

*Disclaimer: This software is in alpha-test mode, 
equivalent to the MESSy red traffic light status.
No unexpected behaviour was observed under testing, and users are 
invited to test with their model setup. However, no express guarantee
is provided for production simulations. 
For assistance or to report problems please contact the maintainers:*
christoudias@cyi.ac.cy; m.alvanos@cyi.ac.cy
 
## 1. Requirements:

Software: CUDA compiler and python are required for the processor. 
Hardware: CUDA compatible GPU. 

## 2. Installation:

There are two files required to enable using the GPUs: 
`f2c_alpha.py`  and `kpp_integrate_cuda_prototype.cu`. 

The files have to be available in the messy/util directory. 
No additional changes are required. 

Note: MESSy has to be linked with the `-lcudart` and `-lstdc++` flags. 
For example, you can append it to the `SPEC_NETCDF_LIB` variable 
in the configuration file (under `config/mh-XXXX`).

## 3. Running the MECCA Fortran to CUDA source-to-source pre-processor:

You have to enter the ./messy/util directory to execute the
preprocessor, by running "`python f2c_alpha.py`". The preprocessor expects
the following files to be in place:

* `messy/smcl/messy_mecca_kpp.f90`
* `messy/smcl/messy_cmn_photol_mem.f90`
* `messy/smcl/messy_main_constants_mem.f90`
* `messy/util/kpp_integrate_cuda_prototype.cu`
* `messy/smcl/specific.mk`
* `messy/smcl/Makefile.m`
 
If any of these files is missing or not configured as in the MESSy release,
the preprocessor will stop with an error message.

## 4. Running EMAC with GPU MECCA and improving performance:

During testing it was found that the runtime parameter `NPROMA` should be set 
to a value not greater than 128 (preferably 64) for optimal memory allocation 
and performance on the GPU.

Each CPU process that offloads to GPU requires a chunk of the GPU VRAM memory,
dependent on the number of species and reaction constants in the MECCA mechanism. 
The number of GPUs per node and VRAM memory available in each GPU dictates the
total number of CPU cores that can run simultaneously.

### NVIDIA Multi-Process Service
To run multiple CPU processes per GPU, the Multi-process service (MPS) provided 
by NVIDIA can be used.

***Warning: Memory Protection***

Volta MPS client processes have fully isolated GPU address spaces. Pre-Volta MPS client 
processes allocate memory from different partitions of the same GPU virtual address space. As a result:

* An out-of-range write in a CUDA Kernel can modify the CUDA-accessible memory state of
another process, and will not trigger an error.
* An out-of-range read in a CUDA Kernel can access CUDA-accessible memory modified by 
another process, and will not trigger an error, leading to undefined behavior.

## Unit tests

A self-contained unit test is included in the ditribution. The test includes 
reference source files implementing a simplified chemistry mechanism and 
compiles, exexutes and compares the FORTRAN (using gfortran) 
and auto-generated CUDA versions.

The test is executed by sourcing `driver.sh` under the `tests` directory. 
A utility script that compares the test solver output is also included in `tests/compare.py`
