# Pixel Streaming on Azure (Linux)


## Important Notes:
This guide is made specifically for Unreal Engine 4.27 and will not work if you’re using any earlier versions.

If you're interested in running an Azure Windows instance, please see the [Pixel Streaming on Azure (Windows)](Pixel%20Streaming%20on%20Azure%20(Windows).md) guide! 

Please check the [FAQ](../FAQ.md) for extra information.

## Initial Requirements:
Working with this guide assumes you have the following:

- An active account with Azure (Azure)
- A packaged, pixel streaming ready application

For more information on this, please familiarize yourself with the following documents:

- Getting Started with Pixel Streaming:
https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/PixelStreaming/PixelStreamingIntro/

- Hosting and Networking Guide
https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/PixelStreaming/Hosting/

- For this guide, you can use the Pixel Streaming Demo provided by Epic
https://docs.unrealengine.com/4.27/en-US/Resources/Showcases/PixelStreamingShowcase/
Or any other Pixel Streaming ready application.


We’ll be using Filezilla to transfer files to the instance, if you don’t wish to use Filezilla you’ll have to use SCP.
https://docs.Azure.amazon.com/AzureEC2/latest/UserGuide/AccessingInstancesLinux.html

This link also describes SSH, which we will be using in this guide.

## Setting up your Azure server

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
For your image, select "See all images", Select AI + Machine Learning on the left panel, scroll down and select `NVIDIA Image for AI using GPUs` > `v21.06.0 -Gen1`.

You should now be back on the Create a virtual machine page. Leave Azure Spot Instance unticked, and make sure the Size category has:
```
Standard_NV6 - 6 vcpus, 56GiB memory
```
We have done the above 2 steps to ensure we are using Nvidia resources. Much like the other guides, it's all to make sure our Pixel Streaming application runs smoothly!

For the Authentication type, set it to SSH. Fill in a suitable username and key pair name.. This will be used to connect to the instance so don't forget these details or you will have to re-create the instance!

You are now done with the basic set up, click Next: Disks.

### **Disks**

You can leave these options at default values. Click Next: Networking.

### **Networking**

You'll see that it has filled in the fields with (new)vmName, you can leave these as is.

Underneath the Configure Network Security Group dropdown bar, you should see `Create new`. Click on that.

Here, you can add ports as required. Alongside the default ones that are there, make sure you have the following ports added.

```
Protocol: Any, Destination port range: 80, Source port ranges: *
```
```
Protocol: Any, Destination port range: 8888, Source port ranges: *
```

Click OK to go back to the Networking window.

Click Next: Management

### **Management**

You can leave these values default, click Next: Advanced.

### **Advanced**

Azure offers an incredibly useful Extensions feature. Unlike the other cloud services, you can simply add the Nvidia requirements pre-launch of the instance.

Click "Select an extension to install"  and look for "NVIDIA GPU Driver Extension", select that and click Next.


Click Next: Tags

### Tags

Skip this step, click Next: Review + Create.

### **Review + Create**

Have a quick read to make sure your settings are correct and once you're satisfied, hit Create.
(Note the hourly price at the top of this window).

It'll take you to an overview screen. Wait until it states "Your deployment is complete. This can take a few minutes.

When it prompts you to download the .pem key file, make sure you do so and store it in a safe location. You will need this file to connect to your instance!

Your instance is now up and running, Congratulations!

Click "Go to resource" to see options for your instance.

## Transferring Pixel Streaming Demo/Application onto the Instance

Launch Filezilla

Select File > Site Manager

In the General tab, fill the following:

- Protocol: SFT - SSH FIle Transfer Protocol
- Host: Your Instances Public IPv4
- Logon Type: Key File
- User: The username you set
- Key file: Browse to your key file
- Connect

You’ll likely get a pop up for “Unknown host key”, you’re fine to click OK
once connected. 

Browse to your zipped application on the left, drag the zip to your desired location on the instance on the right.

Depending on your application and connection speed, this can take a while, but you can move on to the next step while it’s uploading.

## Connecting to Instance via SSH
Open a new terminal

SSH comes pre-installed on some systems, confirm if you have it by typing `ssh` in terminal.

On your instance information page/resource window, press "Connect" at the top left and select SSH.

You'll see a convenient 4 step procedure for connecting to your instance. If you have SSH installed you can skip step 1.

If you copy the example in step 4, replace the "private key path" with your own and put the full command in a terminal window, you will connect to the instance.

Use the following command to connect to your instance (replace the necessary fields with your own information:

```
ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-IP
```

In this case:
- The key pair.pem is the key pair file you downloaded earlier
- The instance user name is the user name you set in the basics segment of the set up.
- I suggest copying the example command on the connect window in step 4, as this will have automatically put the correct username and IP in the command.

It will state that **“The authenticity of host can't be established”**, type yes and press enter.
Wait a moment and the terminal will connect.

You'll know it's finished connecting when the directory in the terminal becomes "UserName@InstanceName"

I recommend opening 2 terminal tabs and connecting twice, it’ll make it easier to navigate and set up your signalling server and application, though this isn’t mandatory. (You won’t have to confirm authenticity in the second window).

Done!

## Preparing Prerequisites

Now that you’re connected to your Azure instance, you’ll need to prepare a few packages so your application can run.
These packages may vary depending on your needs, but for this example we need Vulkan and Pulse Audio. 


### Installing Vulkan
As stated earlier, the Deep Learning instance we’re using already has the required nvidia drivers, but we still need to install Vulkan.
In either of your terminals connected to the instance, type the following: 

```
sudo apt install vulkan-utils
```

Enter Y to confirm

### Installing Pulse Audio

Similar to above, enter the following: 

```
sudo apt-get install -y pulseaudio
```

(This is really only needed if your project has sound. The pixel streaming demo from Epic does not include any audio.)
 

## Preparing Your Files

Now that your files have been transferred to your instance, we can get things moving!

In one of your terminal windows, enter `ls` to see the contents of your current directory.
You should see the zip file of your application.

Type `unzip filename.zip`

Once that's finished, enter `ls` again to make sure the new file is there.

Now you’ll need to move the 2 terminal windows into their relevant directories.
In one terminal window, enter:

```
cd ~/AppFileName/LinuxNoEditor/Samples/PixelStreaming/WebServers/SignallingWebServer/ platform_scripts/bash
```
In the other enter:

```
cd ~/AppFileName/LinuxNoEditor/
```
Make sure to replace `AppFileName` with your unzipped file name!

## Skipping Setup Next Time
Now that you have set up your instance, you can opt to create a custom AMI based on your set up! This will mean the next time you load up your AMI, it'll already have everything ready.
If this interests you, please head over to the [FAQ](FAQ.md) for a walkthrough on how to do this. 

This is definitely not a mandatory step, and if you just want to do a quick test this time and set up something different next time, you can skip this step.

## Setting Up Signalling Server

In the terminal window that’s in the bash directory, enter `ls` to see the directories contents.

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

## Running Application

Open your terminal window that is in the LinuxNoEditor directory.

Type `ls` to check contents, you should see `AppName.sh` in there, alongside other files.

It’s important to run the application with some alterations, so enter the following:
```
 ./AppName.sh -PixelStreamingIP=127.0.0.1 -PixelStreamingPort=8888 -RenderOffscreen -ResX=1920 -ResY=1080 -ForceRes
```
The above commands will run the application, specify it’s local IP, ensure the resolution and render it off screen.
It’s important to render off screen, as it isn’t necessary to display the application on the host instance.
If successful, you should see: 
```
Streamer connected: ::ffff:127.0.0.1 
```
Your application is now up and running!


## Connecting to / Testing your Application

Copy your IPv4 address from your instance and open a new web page.

Enter the IPv4 address into the URL on your web browser.

If everything is working, you should now be connected to your application, streamed live from your instance! 
Have fun playing with your pixel stream!

Try opening multiple different windows and connecting, you’ll see input from one window propagate across all the connected peers.

## Shutting Down Your Pixel Stream + Instance

Once you’re done playing with your shiny new stream, it’s important to close all your running applications and instances if you don’t need them.

Make sure to close your signalling server window. Additionally, as it is rendered off-screen, if you wanted to shut down the running application, you can open task manager > processes and close the running application.

Now that your applications have closed, head back to the Azure website and look for your running instance.

On the resource page (or Virtual Machines page), click Delete at the top. It will ask for confirmation. Click okay and you're done!


[![TensorWorks Logo](../Logo/logo.svg)](https://tensorworks.com.au/)