# Pixel Streaming on Azure (Windows)

## Important Notes:
This guide is made specifically for Unreal Engine 5, other guides may not have the correct information due to script changes.

This guide is made for an Azure Windows instance, whilst also using Windows locally.

If you're interested in running an Azure Linux instance, please see the [Pixel Streaming on Azure (Linux)](Pixel%20Streaming%20on%20Azure%20(Linux).md)

Please check the [FAQ](../FAQ.md) for extra information.

## Initial Requirements:
Working with this guide assumes you have the following:

- An active account with Azure
- A packaged, pixel streaming ready application

For more information on this, please familiarize yourself with the following documents:

- Getting Started with Pixel Streaming:
https://docs.unrealengine.com/5.0/en-US/getting-started-with-pixel-streaming-in-unreal-engine/

- Hosting and Networking Guide
https://docs.unrealengine.com/5.0/en-US/hosting-and-networking-guide-for-pixel-streaming-in-unreal-engine/

- For this guide, you can use the Pixel Streaming Demo provided by Epic
https://docs.unrealengine.com/4.27/en-US/Resources/Showcases/PixelStreamingShowcase/
Or any other Pixel Streaming ready application.

## Setting up Your Azure server


### **Basics**

Log into your Azure account.

Make sure you're on your account home page:
https://portal.azure.com/#home

Select Virtual Machines, then Create > Virtual Machine at the top left.

You should now be on the "Create a virtual machine" page. Thankfully, Azure makes it incredibly easy to create a test environment for Pixel Streaming!

You can leave the Subscription box as your current subscription. For the Resource group, create a new group, you can call this anything you like (I suggest something easy, like training or test).

Give your virtual machine a name, for this example I'll call it GuideTestPS.

Set your Region to the most suitable option. In this example you can just set it to your closest region, but in future, different regions have access to different Images.

Leave the Availability Options and Security type as default values.

**Important Step!*** 
For your image, select `Windows Server 2019 Datacenter - Gen1`. This allows us to use an NV series GPU.

Leave Azure Spot Instance unticked, and make sure the Size category has:
```
Standard_NV6 - 6 vcpus, 56GiB memory
```
You can find and apply this by clicking "See all sizes" and searching for NV. It'll be under the "Non-premium storage VM sizes"

We have done the above 2 steps to ensure we are using Nvidia resources. Much like the other guides, it's all to make sure our Pixel Streaming application runs smoothly!


For the Authentication type fill in a suitable username and password. This will be used to connect to the instance via RDP, so don't forget these details or you will have to re-create the instance!

Make sure "Public inbound ports > Allow selected ports" is selected.
Click the inbound ports drop down, and tick all the port options.

You can skip the licensing step. Leave the box unticked.

You are now done with the basic set up, click Next: Disks.

### **Disks**

You can leave these options at default values. Click Next: Networking.

### **Networking**

You'll see that it has filled in the fields with (new)vmName, you can leave these as is.

You already specified the ports in the basic setup step, so leave this as is.

Click Next: Management

### **Management**

You can leave these values default, click Next: Advanced.

### **Advanced**

Azure offers an incredibly useful Extensions feature. Unlike the other cloud services, you can simply add the Nvidia requirements pre-launch of the instance.

Click "Select an extension to install" and type "Nvidia" in the search bar. You should be presented with a "NVIDIA GPU Driver Extension", select that and click Next.

For other options to install drivers, see their guide here:

https://docs.microsoft.com/en-us/azure/virtual-machines/windows/n-series-driver-setup


Click Next: Tags

### **Tags**

Skip this step, click Next: Review + Create.

### **Review + Create**

Have a quick read to make sure your settings are correct and once you're satisfied, hit Create.
(Note the hourly price at the top of this window).

It'll take you to an overview screen. Wait until it states "Your deployment is complete.

Your instance is now up and running, Congratulations!

Click "Go to resource" to see options for your instance.

## Connecting to Your Instance

Connecting to your Windows Instance is exceptionally easy if you're also running Windows!

In this resource window, you should see "Connect" at the top. Give that a press and select RDP.

Leave the fields as they are, and click "Download RDP File". You don't need to save the file, simply run the file and it'll immediately attempt a connection.

On the resulting window, click on RDP Client, this is where your username and password you made earlier comes in handy!

You should now have a new Windows desktop opened, this is your brand new Windows instance, exciting!

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

Now that your applications have closed, head back to the Azure website and look for your running instance.

On the resource page (or Virtual Machines page), click Delete at the top. It will ask for confirmation. Click okay and you're done!

[![TensorWorks Logo](../Logo/logo.svg)](https://tensorworks.com.au/)