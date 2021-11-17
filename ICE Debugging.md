# Debugging ICE Connections Using Pion/TURN

This guide is designed to help you test your STUN/TURN connectivity using [Pion/TURN](https://github.com/pion/turn/tree/master/examples).

**_Note:_** This turn server is not designed for production. Using this outside of a testing scenario can result in poor performance.

This guide is broken up into the following steps:

1. Setting up Pion/TURN Server on your cloud machine

2. Run a ICE Trickle test using your fresh new Pion/TURN server details

3. Ping your Pion/TURN using the Pion/TURN client ping utility

##  1. Setting up Pion/TURN on your cloud instance

1. Log into your cloud Instance

2. Download and unzip your relevant OS folder from [here](https://github.com/pion/turn/releases/tag/v2.0.2)

3. Open cmd in directory and run:

`turn-server-simple.exe -public-ip $INSTANCE_PUBLIC_IP -users username=password`

## 2. Setting up Ice Trickle Test

1. To set up ice trickle test for new custom turn server, head to: https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/

2. Click Remove server to clear the current default

3. Enter the following into the relevant fields: 

    * Enter turn:publicip:port in the TURN URI category
    * Enter "username" in username field
    * Enter "password" in password field

4. Click add server.

5. Click "Gather candidates"

6. Successful results will show "Relay candidates" 


## 3. Ping your TURN server

1. Unzip the same files from the instance on your local machine

2. Open cmd in the directory

3. Run the following command:

`turn-client-udp.exe -host $INSTANCE_PUBLIC_IP -user=username=password -ping`

4. Successful ping will show standard ping output (e.g. Reply from IP)

For more details about additional commands you can run, please see the provider page here:

https://github.com/pion/turn/tree/master/examples 



[![TensorWorks Logo](Logo/logo.svg)](https://tensorworks.com.au/)