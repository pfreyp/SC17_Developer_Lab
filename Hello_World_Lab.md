# Hello World Lab - SC17

* Navigate to the 'helloworld_ocl' example directory
```
cd /home/centos/SC17_Developer_Lab/helloworld_ocl
```

## Design files

The application source code is located in the src directory. The host and accelerator binary files are pre-compiled and included in xclbin directory.
```
Makefile
src/host.cpp
src/vector_addition.cl
xclbin/hello_world
xclbin/vector_addition.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin
```
## Run both software and hardware emulation

As part of the capabilities available to an application developer, SDAccel includes environments to test the correctness of an application at both a software functional level and a hardware emulated level. These modes, which are named sw_emu and hw_emu, allow the developer to profile and evaluate the performance of a design before compiling for board execution. It is strongly recommended that all applications be checked in at least the sw_emu mode before execution on FPGA accelerator.

* Run the SW Emulation flow

The software emulation flow is a functional correctness check only.

Note that the make file depends on the location of the common repository for includes.
They are expected to be found here: /home/centos/src/project_data/aws-fpga/SDAccel/examples/xilinx.

Edit that path (first line of the make file) if needed.
```
make clean
make check TARGETS=sw_emu DEVICES=$AWS_PLATFORM all
```

* Run the HW Emulation flow
The hardware emulation flow is a cycle accurate simulation of the hardware generated for the application. As such, it is expected for this simulation to take longer time.
```
make clean
make check TARGETS=hw_emu DEVICES=$AWS_PLATFORM all
```

## Run the kernel onto the F1 accelerator

To deploy and execute on F1 you need the host application binary (helloworld) and the AWS FPGA binary (vector_addition.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin).

The .awsxclbin is read by the host application to determine which Amazon FPGA Image (AFI) should be loaded in the FPGA.
Generating the AFI takes time as a full FPGA implementation must take place. For this example, the AFI was created in advance to save time.

* Change directory to access pre-compiled binaries
```
cd ./xclbin
```

* Execute the host application on F1:

```
sudo sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh
chmod u+x helloworld
./helloworld 
```

* The example application output should come out like this:

```
Device/Slot[0] (/dev/xdma0, 0:0:1d.0)
xclProbe found 1 FPGA slots with XDMA driver running
platform Name: Xilinx
Vendor Name : Xilinx
Found Platform
Found Device=xilinx:aws-vu9p-f1:4ddr-xpr-2pr:4.0
XCLBIN File Name: vector_addition
INFO: Importing ./vector_addition.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin
Loading: './vector_addition.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin'
Result =
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42
TEST PASSED
sh-4.2#
```
After succesfully running this design, type the exit command to return to the initial shell
```
exit
```
