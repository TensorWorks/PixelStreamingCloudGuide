
## Important Notes:
This guide is made specifically for Unreal Engine 5, other guides may not have the correct information due to script changes. 

This guide is made for an AWS Windows instance, whilst also using Windows locally.

If you're interested in running an AWS Linux instance, please see the [Pixel Streaming on AWS (Linux)](Pixel%20Streaming%20on%20AWS%20(Linux).md)

Please check the [FAQ](../FAQ.md) for extra information.

## Initial Requirements:
Working with this guide assumes you have the following:

- An active account with Amazon Web Services (AWS)
- A packaged, pixel streaming ready application

For more information on this, please familiarize yourself with the following documents:

- Getting Started with Pixel Streaming:
https://docs.unrealengine.com/5.0/en-US/getting-started-with-pixel-streaming-in-unreal-engine/

- Hosting and Networking Guide
https://docs.unrealengine.com/5.0/en-US/hosting-and-networking-guide-for-pixel-streaming-in-unreal-engine/

- For this guide, you can use the Pixel Streaming Demo provided by Epic
https://docs.unrealengine.com/4.27/en-US/Resources/Showcases/PixelStreamingShowcase/
Or any other Pixel Streaming ready application.


## Setting up Your AWS server
Log into your AWS account.

At the top left, click Services > Compute > EC2

Select Launch Instance

This resulting page is where you will configure all the details of your AWS instance. For this example setup, we'll be fairly open with our security permissions.
The following steps will work through this page and help you to create a solid, basic instance:

### Name and Tags
Create a simple name for your instance. This is optional, but very beneficial if you have multiple instances active.
It may be a good habit to name all your instances, as the list can get rather extensive, depending on your uses for AWS.

### Application and OS Images (Amazon Machine Image)
This is where we select the OS and version of our upcoming instance.

Under the Quick Start category, select Windows. You may get a pop up stating that "Some of your current settings will be changed or removed if you proceed". It's fine to click Confirm Changes here.

With the drop down beneath "Amazon Machine Image (AMI)", select Windows Server 2022 Base.

### Instance Type
Here, we'll select our resources the instance will use. This is a combination of CPU, memory, storage and networking capacity.

Select the drop down and select `g4dn.xlarge`. It may help to search for `g4` in the search bar.
If you're running a more intensive application, or need more processing power, you can select `g4dn.2xlarge`.

### Key Pair
If you've gone through these steps before, simply click the drop down and choose your previously made key pair file. You can now skip the remainder of this step.

To create a key pair file, select "Create new key pair".

In this resulting window, enter a suitable key pair name (It's good to have distinguishable names if you plan to have many different types of instances).

For your private key file format, you may select either, though if you're not using PuTTY I would suggest using .pem.

Hit "Create key pair"

It will immediately download your new key pair file. Keep this file stored securely and do **not** lose this file! If you lose this file you will be unable to use this key pair and will be forced to create a new one.

Make sure your newly created key pair is selected in the drop down.

### Network Settings
Now we'll specify the open ports and connectivity of the instance.

Select Edit in the Network Settings window.

If you have done these steps before and previously created a security group, simply choose "Select existing security group" and choose your previously made option under "Common Security Groups". Move on to the next step.

For this example instance, we're going to create a security group that permits all connections. In future, you'll likely want to be more restrictive in your security group. We can however, use this open security group for testing in future!

You can leave VPC, Subnet and Auto-assign public IP as their default values.

Select "Create security group"

Under security group name, name it accordingly (e.g. "allow-everything")

You can leave the description as default.

Under the "Type" drop down, select `"All Traffic"`. For `"Source Type"`, select `"Anywhere"`

With these values set, move on to the next step.

### Configure Storage
Here we'll specify the amount of storage space on the drive. Free tier customers get up to 30gb, so we'll set for that.

Enter `30` for the GiB value, leave the Root volume as `gp2`.

### Advanced Details
We only need to change one setting in this category.
In order to correctly install the relevant drivers on your instance, we must ensure we have the right permissions.

Next to IAM instance profile, select "Create new IAM Profile"

On the resulting page, click Create Role, this will take you to a creation page.

Select EC2 under *Common Use Cases* and click Next at the bottom of the page.

Filter policies by `AmazonS3ReadOnlyAccess` and select that policy, then click next.

Skip the tags step.

Fill in the review page appropriately, it's important to specify a useful name so you know which IAM you're selecting when you start the instance. For example, you could call this one `"AllowGRIDDrivers"`.

Click create.

You have now created an IAM role! Head back to your Configure Instance Details step and select your new IAM role under the IAM role dropdown menu.

Once this is done, click the "Launch Instance" button on the right. This will create your instance and provide a link to view it!

You'll need to wait a moment for the status check to read "2/2 checks passed" before attempting to connect, as before this it will still be setting itself up. This should not take long.

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

## Setting up Your Signalling Server

Head to your projects WebServers directory, found in:

```
\ProjectName\Windows\ProjectName\Samples\PixelStreaming\WebServers
```

If you had already used the “get_ps_scripts” script in your Engine directory, you should already have all required Pixel Streaming scripts in the above directory. If not, you’ll simply need to run the “get_ps_scripts.bat” script. This will download everything you need.

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
-PixelStreamingURL=ws://127.0.0.1:8888 -RenderOffscreen -AllowPixelStreamingCommands
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

Now that your applications have closed, head back to the AWS website and look for your running instance.

Right click on your instance and click Terminate Instance.

Done!

[![TensorWorks Logo](../Logo/logo.svg)](https://tensorworks.com.au/)