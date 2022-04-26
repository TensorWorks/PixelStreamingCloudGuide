# Pixel Streaming on AWS (Linux)


## Important Notes:
This guide is made specifically for Unreal Engine 4.27 and will not work if you’re using any earlier versions.

If you're interested in running an AWS Windows instance, please see the [Pixel Streaming on AWS (Windows)](Pixel%20Streaming%20on%20AWS%20(Windows).md) guide! 

Please check the [FAQ](FAQ.md) for extra information.

## Initial Requirements:
Working with this guide assumes you have the following:

- An active account with Amazon Web Services (AWS)
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
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html

This link also describes SSH, which we will be using in this guide.

## Setting up your AWS server
Log into your AWS account.

Select Launch a Virtual Machine.

Select your AMI, I recommend the `Deep Learning AMI Ubuntu 18.04`, if you use a different AMI it is your responsibility to install the correct NVIDIA drivers and CUDA Toolkit.

Choose an Instance Type.

Filter by **g4dn**, for the Unreal Pixel Streaming Demo I recommend the **8 vCPUs 32 GiB** option.

For this example, you can skip steps up until step 6.

### Step 6, Configure Security Groups

Leave the existing rule and add 2 new rules:
```
HTTP, TCP, 80, Custom, 0.0.0.0/0,::/0
```
```
Custom TCP, TCP, 8888, Custom, 0.0.0.0/0,::/0
```

Setting `0.0.0.0/0,::/0` will permit all IP addresses to connect to the instance. Please specify any restrictions as needed. For this test, you can safely permit all IP addresses.

Review and launch, make sure to read the warnings before proceeding. Once you're satisfied, hit launch.

Choose either an existing key pair, or create a new key pair.
*Do not lose the key pair file provided!* You can not retrieve it and will have to create a new one if you do.

Head to the View Instances page.

Give the instance time to finish its status checks, you can spend this time by checking the instance details by checking its tick box.
It may be worth copying the following into a doc, as you’ll need these values.

- Public IPv4 DNS
- Public IPv4 Address

You’ll be using the public IPv4 address to connect to the instance application once it’s up and running.

## Transferring Pixel Streaming Demo/Application onto the Instance

Launch Filezilla

Select File > Site Manager

In the General tab, fill the following:

- Protocol: SFT - SSH FIle Transfer Protocol
- Host: Your Instances Public IPv4 DNS
- Logon Type: Key File
- User: ubuntu
- Key file: Browse to your key file
- Connect

You’ll likely get a pop up for “Unknown host key”, you’re fine to click OK
once connected. 

Browse to your zipped application on the left, drag the zip to your desired location on the instance on the right.

Depending on your application and connection speed, this can take a while, but you can move on to the next step while it’s uploading.


## Connecting to Instance via SSH
Open a new terminal

SSH comes pre-installed on some systems, confirm if you have it by typing `ssh` in terminal.

If not, refer back to: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html

Use the following command to connect to your instance (replace the necessary fields with your own information:

```
ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-public-dns-name
```

In this case:
- The key pair.pem is the key pair file you downloaded earlier
- The instance user name is ubuntu
- The public dns is the public IPv4 DNS from your instance

It will state that **“The authenticity of host can't be established”**, type yes and press enter.
Wait a moment and the terminal will connect.

I recommend opening 2 terminal tabs and connecting twice, it’ll make it easier to navigate and set up your signalling server and application, though this isn’t mandatory. (You won’t have to confirm authenticity in the second window).

Done!

## Preparing Prerequisites

Now that you’re connected to your AWS instance, you’ll need to prepare a few packages so your application can run.
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
cd ~/AppFileName/LinuxNoEditor/Samples/PixelStreaming/WebServers/SignallingWebServer/platform_scripts/bash
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


## Shutting Down Your Pixel Stream

Once you’re done playing with your shiny new stream, it’s important to close all your running applications and instances if you don’t need them.

In your terminal window running the application, press CTRL + C
This stops any running processes in that window.
Do the same for your other terminal window running the signalling server.

Now that your applications have closed, head back to the AWS website and look for your running instance.

Right click on your instance and click Terminate Instance.

Done!


[![TensorWorks Logo](../Logo/logo.svg)](https://tensorworks.com.au/)