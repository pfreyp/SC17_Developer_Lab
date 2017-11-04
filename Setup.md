# Lab Setup - SC17

In this section of the Lab, we will be setting up our instance and validating the flow with a simple example.

## Start a preconfigured EC2 F1 instance

For this event, each participant has been attributed an EC2 F1 instance, a user name and password.

The user name and password were communicated through an email message sent a couple of days before the event.
The message contains a URL that will help you access an EC2 F1 instance.
```
If you have not received that email, please contact a staff member at the beginning of the lab session.
```
- Retrieve and navigate to that URL, it points to the AWS EC2 console management page.
- Enter the Account ID communicated by the staff.
- Click "Next"
- Enter the IAM user name received in the email mentioned above and the password.
- Click "Sign In"

You should see one stopped EC2 F1 instance.
Start the instance (click on the "actions" pulldown button and select "start").
Allow some time for the instance to start...
Once the instance is running, the EC2 console gives you information relative to the **public IP address** of the instance.
You will use that IP address for the next step.

## \"Remote desktop\" to the instance

The instance just started is preconfigured with remote desktop protocol (RDP) services.
- From your local machine, start a remote desktop protocol client
   - On Windows: press the Windows key and type "remote desktop".  You should see the "Remote Desktop Connection" show in the list of programs.  (Alternatively you could also directly invoke mstsc.exe)
   - On Linux: any RDP client such a Remmina or Vinagre are suitable
   - On macOS: Microsoft Remote Desktop from the Mac App Store
- In the RDP client, enter the **public IP address** you obtained from the EC2 console page into the 'Computer' text entry field of the client
- Click **Connect**
This should bring up a message about connection certificates. 
- Click **Yes** to proceed.
The Remote Desktop Connection window opens with a login prompt. 
- Log in with the following credentials:
   - User: **user name from the email message**
   - Password: **xilinx_sc17**
- Click **Ok**
You should now be connected to the instance.

## Configure the Xilinx SDAccel environment and load the workshop files

* Open a terminal window, execute the following commands to setup the SDAccel environment (AWS_FPGA_REPO_DIR and XILINX_SDX are predefined)
```  
cd $AWS_FPGA_REPO_DIR                                         
source sdaccel_setup.sh
source $XILINX_SDX/settings64.sh 
```

* Get the SC17 developer lab Github repository to copy the necessary files
```
cd /home/centos
git clone https://github.com/Xilinx/SC17_Developer_Lab
```

## Run a test application

The SDAccel emulation flows allows testing, profiling and debugging the application before deploying on F1. 

* Navigate to the 'helloworld_ocl' example directory
```
cd /home/centos/helloworld_ocl/
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
* Check that the output log of the previous command contains:
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
