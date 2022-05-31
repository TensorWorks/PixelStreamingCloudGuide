# Pixel Streaming on GCP (Linux)

## Important Notes:
This guide is made specifically for Unreal Engine 5, other guides may not have the correct information due to script changes.

This guide is made for a GCP Linux instance.

If you're interested in running an GCP Windows instance, please see the [Pixel Streaming on GCP (Windows)](Pixel%20Streaming%20on%20GCP%20(Windows).md)

Please check the [FAQ](../FAQ.md) for extra information.

## Initial Requirements:
Working with this guide assumes you have the following:

- An active account with GCP
- A packaged, pixel streaming ready application

For more information on this, please familiarize yourself with the following documents:

- Getting Started with Pixel Streaming:
https://docs.unrealengine.com/5.0/en-US/getting-started-with-pixel-streaming-in-unreal-engine/

- Hosting and Networking Guide
https://docs.unrealengine.com/5.0/en-US/hosting-and-networking-guide-for-pixel-streaming-in-unreal-engine/

- For this guide, you can use the Pixel Streaming Demo provided by Epic
https://docs.unrealengine.com/4.27/en-US/Resources/Showcases/PixelStreamingShowcase/
Or any other Pixel Streaming ready application.


## Create a SSH Key

In order to easily transfer files to the instance, or have an extra layer of security, it's a good idea to create an SSH key in advance. If you didn't want to do this step, you can transfer files to the instance using GCPs built in file transfer tool. Keep reading to learn more.

Open a new terminal window and enter the following command:

```
ssh-keygen -t rsa -f ~/.ssh/KEYNAME -C USERNAME -b 2048
```
Simply replace "KEYNAME" with the desired name of the file and USERNAME with your username!

Once done, you'll end up with 2 files in your ~/.ssh directory. One public key and one private key. You'll use these to connect your Filezilla to the instance.

Keep these files safe, as you can use them for every instance going forward.

## Setting up Your GCP server

Head to the VM instances page. This can be found under the 3 line menu at the top left > Compute Engine > VM Instances.

Click "Create Instance"

This window is where we'll specify all of the parameters for our instance.

Set yourself a simple name for the instance. For this test I'll call it GuideTest.

With GCP, it's a bit different and can take a few stabs to get the right setup. As we want to test a pixel streaming application we need to set up a GPU instance. Not every region has access to GPU instances, additionally, they are subject to availability.

For this test, lets choose the following:

- Region: europe-west4(Netherlands)
- Zone: europe-west4-a


### **Machine Configuration**

Machine family: GPU

GPU Type: Nvidia Tesla V100

Number of GPUs: 1

You can leave "Enable Virtual Workstation unticked"

Machine type: n1-standard-2 (2vCPU, 7.5 GB memory)

CPU platform: Automatic

Tick to enable Display device.

### **Confidential VM Service**

Leave this unticked.

### **Container**

Skip this step

### **Boot Disk**

This is the important step in which we specify our operating system!

Hit change and enter the following values:

- Operating System: Linux
- Version: Ubuntu 20.04 LTS
- Boot disk type: Balanced persistent disk
- Size: 50gb

You can leave the advanced configuration default.

### **Identity and API Access**

Leave your Service account as the default. Do this for access scopes as well.

### **Firewall**

Tick both "Allow HTTP Traffic" and "Allow HTTPS Traffic"

### Security

Expand the drop down to see the remaining steps. Click on Security.

You'll see an Add Item button. Press this and you'll be presented with an entry field.

Copy the entire contents of the public ssh key you create early and paste it into field

You can now skip the remaining steps on this window and hit "Create" at the bottom!

You should now see the VM Instances window. This shows you all currently active instances on your account.

It will take a little while for the instance to load, so leave it for a moment until it's all fully up and running.

## Connecting to Your Instance

Under the "Connect" column on your instance, you should see "SSH" with a down arrow.
Click SSH to connect to your instance in your browser! (*__Important Note:__* I highly recommend using Google Chrome. There are some difficulties with pasting commands in the SSH window on other browsers.)


## Transferring Your Application to the Instance

### **Filezilla**
Launch Filezilla

Select File > Site Manager

In the General tab, fill the following:

- Protocol: SFT - SSH FIle Transfer Protocol
- Host: Your Instances Public IP address
- Logon Type: Key File
- User: username of the instance (you can see this in your connected instance window e.g. michael@instance-1:)
- Key file: Browse to your private key file
- Connect

You’ll likely get a pop up for “Unknown host key”, you’re fine to click OK
once connected. 

Browse to your zipped application on the left, drag the zip to your desired location on the instance on the right.

Depending on your application and connection speed, this can take a while, but you can move on to the next step while it’s uploading.

### **GCP Upload**

Alternatively, if you didn't want to set up the SSH key, you can use GCPs built in file transfer. Keep in mind that this is a lot slower than using Filezilla.

Click the cog icon at the top right of the connected instance window and select "Upload file". 
Simply zip your desired application and select it for upload.

## Installing Essentials

GCP Linux instances come with the very bare minimum when it comes to pre-installed packages. This section is to help fill in the blanks!

Run each of the following commands:

```
sudo add-apt-repository ppa:oibaf/graphics-drivers -y

sudo apt update

sudo apt upgrade -y

sudo apt install build-essential -y

sudo apt install unzip

sudo apt install pulseaudio -y

sudo apt install gcc

sudo apt install make

```

## Installing NVIDIA Drivers

Google has thankfully provided easy access information for installing drivers. You can read about it here:
https://cloud.google.com/compute/docs/gpus/install-grid-drivers

On the above page, head down to "Install GRID Drivers" and click on Linux. You should see commands to download and install the drivers. For brevity, I'll provide those commands here!

1. Download the drivers
```
curl -O https://storage.googleapis.com/nvidia-drivers-us-public/GRID/GRID13.0/NVIDIA-Linux-x86_64-470.63.01-grid.run
```
2. Install the drivers
```
sudo bash NVIDIA-Linux-x86_64-470.63.01-grid.run
```

3. During the installation, choose the following options:

If you are prompted to install 32-bit binaries, select Yes.

If you are prompted to modify the x.org file, select No.



## Setting up Your Signalling Server

Change your directory on the instance to your signal server files, the directory will be similar to: `~/LinuxNoEditor/Samples/PixelStreaming/WebServers/SignallingWebServer/platform_scripts/bash`

Now that you're in the bash directory, enter `ls` to see the directories contents.

You should see a variety of `.sh` files, you can always refer to the README in the folder to see what each file does.

Type `./setup.sh`, this will install all the pre-requisites to run a signalling server!

Once that is done type `./Start_WithTURN_SignallingServer.sh` 

If you have done all of the above correctly, you should see:
```
WebSocket listening to Streamer connections on :8888
WebSocket listening to Players connections on :80
Http listening on *: 80
```

If you see the above, then your signalling server is up and running!

## Running Your Application

With your signalling server running, we want to leave that window alone. Click the cog icon at the top right and click "New connection to instancename". This will open a new window to the instance.

Change this new instance window to the LinuxNoEditor directory.

Type `ls` to check contents, you should see `AppName.sh` in there, alongside other files.

It’s important to run the application with some alterations, so enter the following:
```
 ./AppName.sh -PixelStreamingIP=127.0.0.1 -PixelStreamingPort=8888 -RenderOffscreen -ResX=1920 -ResY=1080 -ForceRes -UseHyperThreading
```
The above commands will run the application, specify it’s local IP, ensure the resolution and render it off screen.
It’s important to render off screen, as it isn’t necessary to display the application on the host instance.
If successful, you should see: 
```
Streamer connected: ::ffff:127.0.0.1 
```
Your application is now up and running!

Congratulations! This means your application is running and is connected to your signalling server!

## Connecting to / Testing Your Application

Copy your public IP address from your instance and open a new web page. (You can find this back on the resource page where you first hit connect).

Enter the public IP into the URL on your web browser on your own local machine.

If everything is working, you should now be connected to your application, streamed live from your instance! 
Have fun playing with your pixel stream!

Try opening multiple different windows and connecting, you’ll see input from one window propagate across all the connected peers.

You can connect from any device!


## Shutting Down Your Pixel Stream + Instance

Once you’re done playing with your shiny new stream, it’s important to close all your running applications and instances if you don’t need them.

Make sure to close your signalling server window. Additionally, as it is rendered off-screen, if you wanted to shut down the running application, you can open task manager > processes and close the running application.

Now that your applications have closed, head back to the GCP VM Instances window.

Simply tick the box next to your instance, then click delete at the top of the window. This will take a minute or two. Make sure your instance is deleted before closing your browser!

Done!

[![TensorWorks Logo](../Logo/logo.svg)](https://tensorworks.com.au/)