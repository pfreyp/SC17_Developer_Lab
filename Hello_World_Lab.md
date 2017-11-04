# Hello World Lab - SC17

* Navigate to the 'helloworld_ocl' example directory
```
cd /home/centos/SC17_Developer_Lab/helloworld_ocl
```

## Design files

The application code is located in the src directory. The host and accelerator binary files are pre-compiled and included in the precompiled directory.
```
Makefile
description.json
src/host.cpp
src/vector_addition.cl
precompiled/hello_world
precompiled/vector_addition.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin
```
## Run both software and hardware emulation

As part of the capabilities available to an application developer, SDAccel includes environments to test the correctness of an application at both a software functional level and a hardware emulated level. These modes, which are named sw_emu and hw_emu, allow the developer to profile and evaluate the performance of a design before compiling for board execution. It is strongly recommended that all applications be checked in at least the sw_emu mode before execution on FPGA accelerator.

* Run the SW Emulation flow
The software emulation flow is a functional correctness check only. 
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
Since the creation of that file and AFI are not instantaneous, they have been pre-generated to save time.

* Change directory to access pre-compiled binaries
```
cd ./preconfigured
```

* Confirm that the AFI is ready to be used
Note that in the context of this workshop, the command below might request you might not go through directly.  It might ask you to run 'aws configure' but that inormation might not be available.  If that's the case just move to the next step.
```
aws ec2 describe-fpga-images --fpga-image-ids afi-0f47c8a14a646f4f8
```
* Check that the output contains:
```
...
"State": {
    "Code": "available"
 },
...
```

* Execute the host application on F1:
```
cd ..
sudo sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh   
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
