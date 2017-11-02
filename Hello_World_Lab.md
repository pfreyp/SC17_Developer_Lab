# Hello World Lab - SC17


## Run a test application

The SDAccel emulation flows allows testing, profiling and debugging the application before deploying on F1. 

* Navigate to the 'helloworld_ocl' example directory
```
cd <TBD>/helloworld_ocl/
```

* Run the SW Emulation flow

```
make clean
make check TARGETS=sw_emu DEVICES=$AWS_PLATFORM all
```

* Run the HW Emulation flow

```
make clean
make check TARGETS=hw_emu DEVICES=$AWS_PLATFORM all
```

### Deploy the host application on F1

To deploy and execute on F1 you need the host application (helloworld) and the AWS FPGA binary (vector_addition.hw.xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0.awsxclbin).

The *.awsxclbin is read by the host application to determine which Amazon FPGA Image (AFI) should be loaded in the FPGA.

Since the creation of the .awsxclbin file and AFI are not instantaneous, they have been pre-generated to streamline this workshop.

* Confirm that the AFI is ready to be used
```
aws ec2 describe-fpga-images --fpga-image-ids <AFI ID>
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

* The example application will display the following messages:

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
