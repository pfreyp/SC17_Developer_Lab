# Xilinx Developer Lab - SC17

Welcome to the Xilinx developer lab at SC17.
During this session you will gain hands-on experience with AWS accelerated applications leveraging FPGA.

Let's get started...

## Starting your EC2 F1 instance

For this lab, each participant has been attributed an instance, a user name and a password.

This information has been communicated through an email message sent a couple of days before the event.
If you have not received that email, please contact one of the staff members at the beginning of the lab session.

Retrieve and navigate to the link that points to the AWS EC2 console management page.
You should see a stopped instance.
Start that instance (click on the "actions" pulldown button and select "start").
After a few seconds, the instance should start...
Verify that the EC2 console gives you information relative to the IP addresses as it's necessary for the next step.

## Remote desktop to your instance

The SC'17 F1 instances are preconfigured with remote desktop services (XRDP).
- On your local machine, start a RDP client
   - On Windows: press the start key and start typing "remote desktop".  You should see the "Remote Desktop Connection" show in the list
   - On Linux: any RDP client such a Remmina or Vinagre
   - On MAC: Microsoft Remote Desktop from the Mac App Store
- In the RDP client, put the public IP address of the EC2 instance (usually in the 'Computer' pull down of the client)
- Click **Connect**
This should bring up a message about connection certificates. 
- Click **Yes** to proceed.
The Remote Desktop Connection window opens with a login prompt. 
- Log in with the following credentials:
   - User: centos
   - Password: _password provided by Xilinx staff_
- Click **Ok**
You should now be connected to the instance.

## Run a basic application to confirm the machine is running as expected
