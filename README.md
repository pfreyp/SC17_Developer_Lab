# Xilinx Developer Lab

Welcome to the Xilinx developer lab at SC17.
During this session you will gain hands-on experience with AWS accelerated applications leveraging FPGA.

Let's get started...

## Starting your instance

For this lab, each participant has been attributed an instance and credentials.

The information has been communicated through an email message sent a couple of days before the event.
(If you have not received that email, please contact one of the staff members at the beginning of the lab session)

The email contains a link that points to the AWS EC2 console management page.
Click on that link, you should see a stopped instance.
Now start that instance (click on the "actions" pulldown button and select "start").

After a few seconds, the instance should start...
Verify that thw EC2 console gives you information relative to the IP addresses.

## Remote desktop to your instance

Note that the SC'17 F1 instances have been preconfigured with the XRDP services. If you were to start your own F1 instance, you have to install and start the XRDP services yourself.

- On your local machine, start a RDP client
   - On Windows: mstsc.exe
   - On Linux: any RDP client such a Remmina or Vinagre
   - On MAC: Microsoft Remote Desktop from the Mac App Store
- In the RDP client, put the Public IP address of the EC2 instance (usually in the 'Computer' pull down of the client)
- Click **Connect**
This should bring up a message about connection certificates. 
- Click **Yes** to proceed.
The Remote Desktop Connection window opens with a login prompt. 
- Log in with the following credentials:
   - User: centos
   - Password: _password provided by Xilinx staff_
- Click **Ok**
You should now be connected to the instance.
From that instance, you will retrieve the IP address which will be used to remote desktop into.

If you're using Windows, use "remote desktop"
If you're on Linux use remmina (it might not be installed bydefault on your system)

Once the remore desktop witnowd shows, enter the credentials. that answer record egiven in the email.

At this point you shound be able to access the removte EC2 machine and be presented with a Gnome environment.

Once you are in the enviromnnt, you can got clone this area to retrieve the lab files and instructions.

## Access the lab instructions

## Run a basic application to confirm the machine is running as expected
