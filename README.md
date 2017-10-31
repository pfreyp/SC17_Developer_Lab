# Xilinx Developer Lab - SC17

Welcome to the Xilinx developer lab at SC17.
During this session you will gain hands-on experience with AWS accelerated applications leveraging FPGA.

Let's get started...

## Starting your EC2 F1 instance

For this lab, each participant has been attributed an instance, a user name and a password.

The user name and password were communicated through an email message sent a couple of days before the event.
That message also contained a URL that points to an instance that you will need to start.
If you have not received that email, please contact one of the staff members at the beginning of the lab session.

Retrieve and navigate to that URL, it points to the AWS EC2 console management page.
You should see one stopped EC2 instance.
Start that instance (click on the "actions" pulldown button and select "start").
After a few seconds, the instance should start...
Verify that the EC2 console gives you information relative to the IP addresses as it's necessary for the next step.

## Remote desktop to your instance

The SC17 F1 instances are preconfigured with remote desktop protocol (RDP) services.
- From your local machine, start a remote desktop protocol client
   - On Windows: press the start key and type "remote desktop".  You should see the "Remote Desktop Connection" program show in the list
   - On Linux: any RDP client such a Remmina or Vinagre
   - On MAC: Microsoft Remote Desktop from the Mac App Store
- In the RDP client, enter the public IP address you read from the EC2 console page into the 'Computer' text entry field of the client
- Click **Connect**
This should bring up a message about connection certificates. 
- Click **Yes** to proceed.
The Remote Desktop Connection window opens with a login prompt. 
- Log in with the following credentials:
   - User: centos
   - Password: _password provided by Xilinx staff_
- Click **Ok**
You should now be connected to the instance.

## Run the 'Hello World' example

### Run the emulation flows

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
