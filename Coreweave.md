## Setting up CoreWeave

Before you begin, we suggest following the [official CoreWeave documentation](https://docs.coreweave.com/coreweave-kubernetes/getting-started) for getting started with your CoreWeave account and setting up your Kubernetes. It should cover all the basic set up and information for preparing your CoreWeave account and Kubernetes.

## Clients

### Lens vs OpenLens

Currently, there are two go-to programs for interacting and tracking your clusters, Lens and OpenLens.

Both are great options, but have variances between them that are worth noting:

1. Lens is built on top of OpenLens, thus containing some additional software and libraries with different licenses.
2. OpenLens is fully open source and has the relevant freedoms.
3. Lens was freeware, but now has a new subscription model. It is free for personal use, but not for larger businesses (this was implemented July 28th 2022, but a grace period until January 2023 was given if you had already been a user).

For information on which software you should go with, it's recommended to research them further. For the sake of simplicity, we'll use OpenLens in this guide.

### OpenLens

1. OpenLens is the open source project that contains the code which supports the main functionality of Lens. Lens is built on top of OpenLens.
2. Head to the [OpenLens Github](https://github.com/MuhammedKalkan/OpenLens) and follow the provided steps to install on your OS. 

**Important note!** There is currently an issue on Windows with OpenLens 6.3.0 that prevents you from seeing your connected cluster details.
Head to the [Releases page](https://github.com/MuhammedKalkan/OpenLens/releases) and download 6.2.5 instead. This will circumvent the issue. Latest is fine if you're using Linux.

For extra information on CoreWeave, you can refer to their official documentation [here](https://docs.coreweave.com/).

### Set up Config

Now that we have Lens and Kubectl set up, we need to set up our config file to use our clusters.

*Note:* You'll need to have set up your own cluster. This can be on a number of platforms, but for this documentation, we're assuming you've set up your cluster on CoreWeave.

There is a lot of information involved with setting up your config file, so we recommend reading up on it here:

1. [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
2. [Organise Cluster Access using Config Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
3. [Setting up Config for Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

Once you've read the guides, you'll need to:

1. On Windows, navigate to `C:\Users\user\.kube`; 

2. On Linux navigate to `/home/user/.config/OpenLens/kubeconfigs/`;

3. Grab the pre-configured config file;

4. Overwrite the default config file with the new one.

Once done with the above steps, it may be worth restarting your system.

### Check Config
1. Open Terminal/Powershell and enter `kubectl config get-contexts`;
2. If set up correctly, your output should show a list of each elements under “Namespace, Cluster and AuthInfo”, including but not limited to, “coreweave-admiral”, “tenant-tw-admiral” and "tenant-tensorworks-testing”

### Using Lens/OpenLens
1. Open Lens and click on the catalog tab at the top left. You should see a variety of Kubernetes clusters available;
2. Click on a cluster to connect. You should be presented with a variety of status categories, such as pods, deployments and storage.

This will be your go-to for keeping track of your pods and applications you have running on CoreWeave.

For details on uploading and running your own applications, continue reading.

## Getting a packaged project onto CoreWeave

Before doing the following, you'll need to set up a Pixel Streaming compatible project in Unreal Engine. If you're unsure how to do this, the Unreal Engine documentation can walk you through the steps [here](https://docs.unrealengine.com/5.1/en-US/getting-started-with-pixel-streaming-in-unreal-engine/). Ensure your project has been packaged for Linux.

### Uploading your Project Data
1. Set up kube config files, making sure they're in the right place so that kubectl will find them. As long as kubectl knows where they are then Lens will too.
2. Switch to the correct CoreWeave context:
```sh
kubectl config use-context coreweave
```
3. Get a copy of [this yaml file](/assets/plaintext/content/sps/coreweave/data-copy.yml)
4. Copy the _stuff_ over using data, preferably an already packaged linux project.
```sh
kubectl cp ./Linux-LockFix data-copy:/data/
```

### Running the Project
While that uploads, you can prepare the yaml file for for actually running the project. You can either make one from scratch, or make a copy of [this example](/assets/plaintext/content/sps/coreweave/third-person-ue51.yml) and modify it to suit your needs.

Once your data-copy pod has finished copying your project content onto the server, apply your new yaml file to get it running:
```sh
kubectl apply -f  third-person-ue51.yml
```

To connect to the running Pixel Stream, you can find the public IP address by running:
```
kubectl get svc third-person-ue51.yml
```
Enter the external IP address from the output into a web browser and you'll connect to the stream!

### Tips and Tricks
* Don't forget that you need a second container for running the signalling server process. This is already included in the example file.
* Once you have a pod running on CoreWeave, you can access its logs and get an SSH terminal to it using OpenLens. Just go to the list of pods on your current context and click the three dots on the far right of the pod you're interested in to open up a live shell or view logs.
* Keep in mind that logs are for the current running instance. If a pod fails and stops for any reason it will typically restart immediately. If you want to see a crash report or something in the log of a previously running instance, you can click "Show previous terminated container" in the log window, however, it will only have the data for one previous instance.

## Debugging on CoreWeave with gdbserver

You'll need to make sure your project's yaml file is going to actually use `gdb` when launching the project instead of the project itself. It should be relatively easy to just [take this example](/assets/plaintext/content/sps/coreweave/third-person-ue51.gdb.yml) and modify it to suit. If making your own, keep the following in mind:
* Make sure the Docker image the pod is starting with installs `gdb` as part of its Dockerfile. `gdb` is not essential software for Ubuntu-based images and will not be present if you don't install it explicitly. Dockerhub has an image `tensorworks/runtime-gdbserver`, which is essentially equivalent to `ghcr.io/epicgames/unreal-engine:runtime-pixel-streaming` but with `gdb` installed.
* You need to ensure the project container is running `gdbserver` instead of the project launch script.
* Additionally, `gdbserver` itself needs to be passed the path to the actual binary. Unlike `gdb` proper, `gdbserver` will try to debug the shell running the launch script, which will put you in a forking hell as you try to figure out which forked process is the one actually running the project.

Once the debugging yaml file is ready to go, upload it to CoreWeave:
```sh
kubectl apply -f  third-person-ue51.gdb.yml
```

You're now ready to connect a debugger and get cracking.

### Using GDB directly from the Terminal
Command-line debugging is something of an acquired taste, but if you're comfortable with your terminal emulator, you can cut out the intermediary and use GDB directly. The big benefit is that you can interrupt whatever `gdb` is doing and issue commands straight to it to do things like loading symbols from local files and printing arbitrary expressions to get their result when evaluated at the current line.

Ensure you're running `gdb` from the root folder of the packaged project. It should be the same as what you uploaded to `/home/ue4/project` with the data-copy pod before.

Before you actually connect to the remote server, run the following to load debugging symbols from your local project:
```gdb
file ProjectName/Binaries/Linux/ProjectName
file ProjectName/Binaries/Linux/ProjectName.debug
```
Doing the first will sometimes trigger the second automatically. You _can_ wait for `gdb` to get symbols remotely over TCP and it will do so for any new libs encountered, but it takes **forever** with UE in particular because a typical packaged project will have some 2.3 GB of symbols to load.

You can also try to get engine symbols as well, though make sure the engine build you're loading here is the same as the one you used to package your project:
```gdb
file /absolute/path/to/UnrealEngine/Engine/Binaries/Linux/UnrealEditor
```

 Enter the following (replacing IP and Port as necessary) to connect your local `gdb` to the pod's running instance:
```gdb
(gdb) target remote 216.153.48.252:2345
```

Once `gdb` connects successfully you'll immediately hit the starting breakpoint, looking something like:
```gdb
0x00007fc0a4526090 in ?? ()
    (gdb)
```
Now you're ready to get started. Enter `continue` (or just `c`) and `gdbserver` will start the project running.

#### Tips & Tricks

* [Here is a cheat-sheet](https://www.utm.edu/staff/jguerin/resources/utilities/gdb.html) of the commands available. More information is available in the [full documentation](https://ftp.gnu.org/old-gnu/Manuals/gdb/html_node/gdb_toc.html). The documentation is usually locally viewable with `info gdb`, though the versions hosted online are much more usable.
* You can put all your `(gdb)` commands in a `.gdbinit` into your local project root as well. When you launch `gdb` from that directory, it will run everything in that file automatically as if you typed it into the prompt.
* Remember that you have to press Ctrl+C in the terminal if you want to pause. If you're trying to catch a lockup, you have to make sure you do so before UE detects the freeze and kills the rendering thread. You have about 60 seconds but it does mean you can't just walk away and get a coffee while you wait for your repro.
* When first starting, it can be hard to tell when it's actually started, as there's a bunch of other libs you have to load symbols for. When `gdb` stops notifying you of them for a good 15-20 seconds, you're probably good to go.
* If it looks like it just won't stop loading symbols (or if you start seeing a library loaded multiple times) interrupt with ^C and run `(gdb) set auto-solib-add off` to just stop it loading library symbols in general.

### Using VSCode for remote debugging
You can create a launch.json entry like this to perform remote debugging:
```json
{
    "name": "RemoteDebug",
    "type": "cppdbg",
    "request": "launch",
    "program": "/LOCAL/PATH/TO/EXECUTABLE",
    "cwd": "/LOCAL/PATH/TO/PROJECT/DIR",
    "MIMode": "gdb",
    "miDebuggerServerAddress": "216.153.48.252:2345",
    "stopAtEntry": false
}
```
