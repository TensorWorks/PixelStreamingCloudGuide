# Pixel Streaming on AWS (Windows)

## Important Notes:
This guide is made specifically for Unreal Engine 4.27 and will not work if you’re using any earlier versions.

This guide is made for an AWS Windows instance, whilst also using Windows locally.

If you're interested in running an AWS Linux instance, please see the [Pixel Streaming on AWS (Linux)](Pixel%20Streaming%20on%20AWS%20(Linux).md)

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
or any other Pixel Streaming ready application.

## Setting up Your AWS server
Log into your AWS account.

Select Launch a Virtual Machine.

Select your AMI, I recommend the `Microsoft Windows Server 2019 Base` for stability purposes.

Choose an Instance Type.

Filter by **g4dn**, for the Unreal Pixel Streaming Demo I recommend the **8 vCPUs 32 GiB** option.

You can skip step 2 and focus on step 3.

### Step 3, Configure Instance Details

For a Windows instance, it's important to specify an *IAM Role*. Having this role set will allow you to install required drivers on your instance.
Click on `Create new IAM role`.

On the resulting page, click Create Role, this will take you to a creation page.

Select EC2 under *Common Use Cases* and click Next at the bottom of the page.

Filter policies by `AmazonS3ReadOnlyAccess` and select that policy, then click next.

Skip the tags step.

Fill in the review page appropriately, it's important to specify a useful name so you know which IAM you're selecting when you start the instance. For example, you could call this one "AllowGRIDDrivers".

Click create.

You have now created an IAM role! Head back to your Configure Instance Details step and select your new IAM role under the IAM role dropdown menu.

You can now safely click next until step 6.

### Step 6, Configure Security Groups

Leave the existing rules and add 2 new rules:
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

## Connecting to Your Instance

Connecting to your Windows Instance is exceptionally easy if you're also running Windows!

In your view instances window, right-click on your instance and click connect.

On the resulting window, click on RDP Client, this is where your keyfile you made earlier (or already had) comes in handy!
Click Get Password, and it will prompt you for the key file. Once you've either provided the file or its contents, you'll have your password.

Open the start menu on your desktop, and type "Remote Desktop Connection". Click on that.

In the resulting pop-up, provide the Public DNS shown on your RDP client webpage then press connect.

Now you simply put in the User name provided to you, as well as the password and you're good to go!

You should now have a new Windows desktop opened, this is your brand new Windows instance, exciting!

## Transferring Your Application to the Instance

One fantastic feature of RDP is the use of your basic "copy paste" you would use anywhere!

Zip your application folder, then CTRL + C the zipped folder. Click back into your instance and CTRL + V on the desktop. Your file will transfer to the instance!

Note: You can move on to the next step while it transfers.

Alternatively, if your file is particularly massive, you might want to save it to a cloud storage medium and download it to the instance. This way means you are less likely to have issues if the transfer gets interrupted.

Extract the application to your desire location on the instance and you're ready to go.

## Preparing for Connections/Installations

Before we can connect to the running stream + install pre-requisites, there are a few default settings in the way.

Open the start menu on your instance, and open "Server Manager".

At the top left of the window, click on "Local Server"

You'll see the properties section displaying the instances security settings. You'll need to change the following:

* Windows Defender Firewall: click and disable all the firewalls in the resulting window.
* Windows Defender Antivirus: click and disable "Real-time protection".
* IE Enhance Security Configuration: click and turn off for both Administrators and Users.

The above settings stop you from installing a lot of the required programs below, as well as prevent you from connecting to your pixel stream later.

## Installing Nvidia Drivers

### Installing GRID Drivers
We generally recommend installing the GRID drivers, as in our experience they are more stable for these use cases than the standard public drivers.

Installing your Nvidia drivers ensures you can properly run the application. Thanks to the AMI role you created and assigned to the instance, we can simply run a powershell command to acquire the necessary drivers!

1. Press Start, and open powershell
2. Copy and paste the following into your powershell window:
 ```
$Bucket = "ec2-windows-nvidia-drivers"
$KeyPrefix = "latest"
$LocalPath = "$home\Desktop\NVIDIA"
$Objects = Get-S3Object -BucketName $Bucket -KeyPrefix $KeyPrefix -Region us-east-1
foreach ($Object in $Objects) {
    $LocalFileName = $Object.Key
    if ($LocalFileName -ne '' -and $Object.Size -ne 0) {
        $LocalFilePath = Join-Path $LocalPath $LocalFileName
        Copy-S3Object -BucketName $Bucket -Key $Object.Key -LocalFile $LocalFilePath -Region us-east-1
    }
}
```
3. Once this is done downloading, run the installer in the new Nvidia file on your desktop
4. Simply following the installer directions and you are done! (express installation is fine)

### Installing Public Drivers

Alternatively, if you wish to install the public drivers found on the Nvidia website, you can head to "http://www.nvidia.com/Download/Find.aspx" and find the correct driver.

For this scenario, as we are running a "g4dn" instance, the correct driver is as follows:

* Product Type: Tesla
* Product Series: T-Series
* Product: T4

***Make sure you get the 426.00 version or later!***

## Installing NPM + Chocolatey

One last pre-requisite until we're up and running!

The signalling server is dependant on chocolatey and node.js to set up, but thankfully they can be installed together.

Head to: https://nodejs.org/en/download/ on your instance, and download the windows installer.

Proceed through the installer and follow the steps.

When you get to the Tools for Native Modules step, it'll ask if you want to "Automaticcaly install the necessary tools. Note, this will also install Chocolatey." Tick the box and click next.

Once node has finished installing, a command window will pop up, prompting to install chocolatey. Press any key and let powershell do its thing. This may take a few minutes.

## Setting up Your Signalling Server

Head to your projects cmd directory, found in:

```
WindowsNoEditor\Samples\PixelStreaming\WebServers\SignallingWebServer\platform_scripts\cmd
```

You'll see a variety of files in here, you can always refer to the README in this folder to see what each of them do. But our first step is to install the pre-requisites to run a signalling server!

1. Open your start menu and search for Powershell
2. Right click Windows Powershell and run as administrator
3. In your powershell window enter `cd PATH\WindowsNoEditor\Samples\PixelStreaming\WebServers\SignallingWebServer\platform_scripts\cmd` (Replacing PATH with your own file path)
4. Enter `.\setup.ps1` 

This should automatically install all the required components to run your signalling server. 
Furthermore, now that you have done this, you do not need powershell to be elevated, so to run the other scripts, you can simply right click them and select "Run with Powershell"

Go ahead and run `Start_WithTURN_SignallingServer.ps1`.
This will open your signalling server window. Once that is done, the output should end in:

```
00:36:55.279 WebSocket listening to Streamer connections on :8888
00:36:55.279 WebSocket listening to Players connections on :80
00:36:55.280 Http listening on *: 80
```

Additionally, you should also see your turnserver.exe window. 

With all of this set up, you now have a signalling server ready!

## Running Your Application

Before we can run the application, we need to set some commands on the executable.

Head back to your applications folder where the .exe is found (default folder is "WindowsNoEditor"). Hold down ALT and click drag the .exe into the same folder. This will create a shortcut.

Right click on the shortcut, and copy paste the following onto the *end* of the target parameter:

```
-PixelStreamingIP=127.0.0.1 -PixelStreamingPort=8888 -RenderOffscreen -ResX=1920 -ResY=1080 -ForceRes
```

This forces the application to render off-screen, which is vitally important when cloud streaming. Failing to do so will cause the instance to use more resources rendering the application locally, rather than on the stream.

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

Now that your applications have closed, head back to the AWS website and look for your running instance.

Right click on your instance and click Terminate Instance.

Done!

[![TensorWorks Logo](Logo/logo.svg)](https://tensorworks.com.au/)