# Pixel Streaming on GCP (Windows)

## Important Notes:
This guide is made specifically for Unreal Engine 5, other guides may not have the correct information due to script changes.

This guide is made for an GCP Windows instance, whilst also using Windows locally.

If you're interested in running an GCP Linux instance, please see the [Pixel Streaming on GCP (Linux)](Pixel%20Streaming%20on%20GCP%20(Linux).md)

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

- Operating System: Windows Server
- Version: WIndows Server 2019 Datacenter
- Boot disk type: Balanced persistent disk
- Size: 50gb

You can leave the advanced configuration default.

### **Identity and API Access**

Leave your Service account as the default. Do this for access scopes as well.

### **Firewall**

Tick both "Allow HTTP Traffic" and "Allow HTTPS Traffic"

You can now skip the remaining steps on this window and hit "Create" at the bottom!

You should now see the VM Instances window. This shows you all currently active instances on your account.

It will take a little while for the instance to load, so leave it for a moment until it's all fully up and running.

## Connecting to Your Instance

Under the "Connect" column on your instance, you should see "RDP" with a down arrow.

Hit that down arrow, and select "Set Windows Password". You can make the username whatever you like. Once this completes, it'll provide you with its randomly generated password. *Do not lose this!* You'll be forced to regenerate the password if you do.

Now that that is set, head back and click on RDP. You'll see a link on the resulting window labelled "Download the RDP file if you will be using a 3rd-party client." Click that link.

You'll immediately be prompted to down or open the file, click Open with and click OK (The open with option should be Remote Desktop Connection by default).

Simply follow the prompts to connect to your instance! You may get a notice saying the certificate is not from a trusted authority, you're fine to ignore this. Enter your password when prompted and you should be connected!


## Transferring Your Application to the Instance

One fantastic feature of RDP is the use of your basic "copy paste" you would use anywhere!

Zip your application folder, then CTRL + C the zipped folder. Click back into your instance and CTRL + V on the desktop. Your file will transfer to the instance!

Note: You can move on to the next step while it transfers.

Alternatively, if your file is particularly massive, you might want to save it to a cloud storage medium and download it to the instance. This way means you are less likely to have issues if the transfer gets interrupted.

Extract the application to your desired location on the instance and you're ready to go.

## Preparing for Connections/Installations

Before we can connect to the running stream + install pre-requisites, there are a few default settings in the way.

Open the start menu on your instance, and open "Server Manager".

At the top left of the window, click on "Local Server"

You'll see the properties section displaying the instances security settings. You'll need to change the following:

* Windows Defender Firewall: click and disable all the firewalls in the resulting window.
* Windows Defender Antivirus: click and disable "Real-time protection".
* IE Enhance Security Configuration: click and turn off for both Administrators and Users.

The above settings stop you from installing a lot of the required programs below, as well as prevent you from connecting to your pixel stream later.

## Installing NVIDIA Drivers

Google has thankfully provided easy access information for installing drivers. You can read about it here:
https://cloud.google.com/compute/docs/gpus/install-grid-drivers

On the above page, head down to "Install GRID Drivers" and click on Windows Server. You should see a link labelled "Windows Server 2016 and 2019" Click that link to be provided the driver installer!

I suggest opening the link directly on your instance, to save the hassle of transferring the file afterward.

Once you've downloaded the file, simply follow the basic installation prompts and you are done! (Express installation should be fine for this example).

## Installing NPM + Chocolatey

One last pre-requisite until we're up and running!

The signalling server is dependant on chocolatey and node.js to set up, but thankfully they can be installed together.

Head to: https://nodejs.org/en/download/ on your instance, and download the windows installer.

Proceed through the installer and follow the steps.

When you get to the Tools for Native Modules step, it'll ask if you want to "Automatically install the necessary tools. Note, this will also install Chocolatey." Tick the box and click next.

Once node has finished installing, a command window will pop up, prompting to install chocolatey. Press any key and let powershell do its thing. This may take a few minutes.

## Setting up Your Signalling Server

Head to your projects cmd directory, found in:

```
WindowsNoEditor\Samples\PixelStreaming\WebServers\SignallingWebServer\platform_scripts\cmd
```

You'll see a variety of files in here, you can always refer to the README in this folder to see what each of them do. But our first step is to install the pre-requisites to run a signalling server!

With UE5, we've massively simplified the setup for your signalling server. Simply double-click `setup.bat` and it will automatically install CoTURN and the node prerequisites in this directory. If setup has worked as normal, you'll now see a coturn and node file in this directory.

Now that this is done, go ahead and run `Start_WithTURN_SignallingServer.ps1`. (Note, you can run powershell scripts simply by right clicking them and selecting "Run with Powershell")

This will open your signalling server window. Once that is done, the output should end in:

```
00:36:55.279 WebSocket listening to Streamer connections on :8888
00:36:55.279 WebSocket listening to Players connections on :80
00:36:55.280 Http listening on *: 80
```

Additionally, you should also see your turnserver.exe window. 

With all of this set up, you now have a signalling server ready, with a TURN connection ready in case STUN fails!

## Running Your Application

Before we can run the application, we need to set some commands on the executable.

Head back to your applications folder where the .exe is found (default folder is "WindowsNoEditor"). Hold down ALT and click drag the .exe into the same folder. This will create a shortcut.

Right click on the shortcut, and copy paste the following onto the *end* of the target parameter:

```
-PixelStreamingIP=127.0.0.1 -PixelStreamingPort=8888 -RenderOffscreen -ResX=1920 -ResY=1080 -ForceRes
```

This forces the application to render off-screen, which is vitally important when cloud streaming. Failing to do so will cause the instance to use more resources rendering the application locally, rather than only on the stream.

Once you've applied the commands to the target, run your shortcut.

You'll likely get a pop up stating:
```
The following components are required to run this program:

DirectX Runtime

Would you like to install them now?
```
Click yes, and go through the installer. Note, it will fail to install .NET Framework 3.5. You can ignore this.

Once that installer is closed, you may notice that your signalling server window shows
```
Streamer connected: ::ffff:127.0.0.1
```

Congratulations! This means your application is running and is connected to your signalling server!

## Connecting to / Testing Your Application

Copy your IPv4 address from your instance and open a new web page.

Enter the IPv4 address into the URL on your web browser on your own local machine.

If everything is working, you should now be connected to your application, streamed live from your instance! 
Have fun playing with your pixel stream!

Try opening multiple different windows and connecting, you’ll see input from one window propagate across all the connected peers.

You can connect from any device!

If you ever wish to simply test it on the instance itself, you can put `127.0.0.1` in a web browser on the instance. I would recommend using Google Chrome for this.


## Shutting Down Your Pixel Stream + Instance

Once you’re done playing with your shiny new stream, it’s important to close all your running applications and instances if you don’t need them.

Make sure to close your signalling server window. Additionally, as it is rendered off-screen, if you wanted to shut down the running application, you can open task manager > processes and close the running application.

Now that your applications have closed, head back to the GCP VM Instances window.

Simply tick the box next to your instance, then click delete at the top of the window. This will take a minute or two. Make sure your instance is deleted before closing your browser!

Done!

[![TensorWorks Logo](../Logo/logo.svg)](https://tensorworks.com.au/)
