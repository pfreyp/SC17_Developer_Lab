<table style="width:100%">
  <tr>
    <th width="100%" colspan="5"><h2>SC17 Xilinx Developer Lab</h2></th>
  </tr>
  <tr>
    <td width="20%" align="center"><b>Introduction</b></td>
    <td width="20%" align="center"><a href="SETUP.md">1. Connecting to your F1 instance</a></td> 
    <td width="20%" align="center"><a href="FFMPEG_Lab.md">2. Experiencing F1 acceleration</a></td>
    <td width="20%" align="center"><a href="IDCT_Lab.md">3. Developing F1 applications</a></td>
    <td width="20%" align="center"><a href="WRAP_UP.md">4. Wrapping-up</td>
  </tr>
</table>

---------------------------------------
# Connecting to your F1 instance

In this section, you will connect to your instance then download the Lab files and instructions.

## Start a preconfigured EC2 F1 instance

For this event, each participant has been attributed an EC2 F1 instance, and login credentials.

You should have received an email that provides you with an account ID, a user name ("user" followed by a number) and a web link to access an EC2 F1 instance.
```
If you have not received that email, please contact a staff member at the beginning of the lab session.
```
- Click on that link, it points to the AWS EC2 console management page.
- Enter the **Account ID** from the email.
- Click **Next**.
- Enter the IAM **user** from the email.
- Enter the password communicated by the staff.
- Click **Sign In**.

You should see one stopped EC2 F1 instance.
- Start the instance by choosing the **Actions** button, then select **Instance State** and then **Start**.

![Start](/setupFigures/start1.png?raw=true)

Allow some time for the instance to start. If needed, click the **Refresh** icon (![Refresh](/setupFigures/refresh2.png?raw=true)) in the top-right corner of the EC2 dashboard to update the instance status information.

Once the instance is running, the EC2 console gives you information relative to the **public IP address** of the instance which we will be using in the next step.

## \"Remote desktop\" to the instance

The instance just started is preconfigured with remote desktop protocol (RDP) services.
- From your local machine, start a remote desktop protocol client
   - On Windows: press the Windows key and type "remote desktop".  You should see the "Remote Desktop Connection" show in the list of programs.  Alternatively you could also directly invoke mstsc.exe.
  
   - On Linux: any RDP client such a Remmina or Vinagre are suitable
   - On macOS: Microsoft Remote Desktop from the Mac App Store
- Set your remote desktop client to use **24-bit for color depth** (Option->Display tab for Windows Remote Desktop).
- In the RDP client, enter the **public IP address** you see in the lower part of the AWS Console Management web page in the "Description" tab
- Click **Connect**
This should bring up a message about connection certificates. 
- Click **Yes** to dismiss the "certificate" window.
The Remote Desktop Connection window opens with a login prompt. 
- **Login** with the following credentials:
   - User: **centos**
   - Password: ******** _(provided at the event)_
   
    ![Remote](/setupFigures/remote1.png?raw=true)
   
- Click **Ok**
You should now be connected to the instance and see a Gnome desktop.

## Configure the Xilinx SDAccel environment and load the workshop files

* Open a terminal window (right mouse click and select "Open Terminal")
* Double click on the Chromium browser icon, it opens to the Lab instructions (if a "keyring" popup comes up, click **Cancel**).  We suggest you **perform all your copy-paste from instructions to shell within the RDP session** to avoid issues.

* In a **shell**, execute the following commands to setup the SDAccel environment and get the necessary files (copy-paste the whole block of commands below in the shell if you'd like)
```  
cd /home/centos
git clone https://github.com/Xilinx/SC17_Developer_Lab.git
export AWS_FPGA_REPO_DIR=/home/centos/aws-fpga
cd $AWS_FPGA_REPO_DIR
source sdaccel_setup.sh
source $XILINX_SDX/settings64.sh 
```

## Running the hello_world example to check the F1 instance

* The hello world example is a vector addition OpenCL example for which we will compile the host code and use a pre-compiled FPGA image.
```
cd ~/SC17_Developer_Lab/helloworld_ocl
make TARGETS=hw DEVICES=$AWS_PLATFORM exe
sudo sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh
./helloworld
```

* A successful outcome would look like the following:
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

We just compiled and executed an F1 application reusing a pre-registered AFI (Amazon FPGA Image).

* Close your terminal (type exit twice)

This concludes the setup Lab.

---------------------------------------

<p align="center"><b>
Start the next module: <a href="FFMPEG_Lab.md">2: Experiencing F1 acceleration</a>
</b></p>
