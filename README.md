# easy-sdo-oh

This project is glue code that I use for Secure Device Onboarding (SDO) with Open Horizon. I wrote this because I have difficulty following the [official Open Horizon documentation for SDO](https://github.com/open-horizon/SDO-support). I hope you will find this helpful too, but if you run into troubles please go to the official documentation and follow the procedures there.it

There is an accompanying [15 minute YouTube video](https://www.youtube.com/watch?v=dNGv2xVVAvs&list=PLgohd895XSUddtseFy4HxCqTqqlYfW8Ix&index=13) you can watch that shows me going through these steps and providing additional explanation. There is also a [5 minute YouTube video](https://www.youtube.com/watch?v=dNGv2xVVAvs&list=PLgohd895XSUddtseFy4HxCqTqqlYfW8Ix&index=2) that gives an overview of how the SDO feature works in Open Horizon.

### Assumptions:

1. You have a Linux device that you would like to setup as an SDO device (i.e., you wish to simulate the manufacture of an SDO device, like those manufactured by Intel and other vendors).

2. You have **user level** credentials for an Open Horizon Management Hub, and you have `HZN_ORG_ID` and `HZN_EXCHANGE_USER_AUTH` configured appropriately on some machine other than the one where you wish to simulate SDO manufactturing.

### Simulate the SDO Manufacturing Steps On Your Device

* login to the machine you wish to make into an SDO device (via ssh, or on the machine console, etc.)

* either `git clone ...` this repo onto that machine, or simply copy the `SDO_DEVICE_STEPS` file onto this machine (as I do in the accompanying video).

* **optionally** you may choose to use a different SDO Rendezvous Server. The example here uses the Intel SDO Rendezvous Server (which will work for you). If you wish, you can change this in the `SDO_DEVICE_STEPS` file. There are more details on this in that file. One option available to all Open Horizon users is to use the SDO Rendezvous Server built into every Open Horizon Management Hub. But again, this is optional. You can use the Intel server.

* when you are ready, run the command below on this machine:

```
source SDO_DEVICE_STEPS
```

* after it has set things up, it will ask you to run a long command involving `simulate-mfg.sh`. Copy that entire long string then paste it and run it. It will take just a few minutes to run (to simulate the process of manufacturing this machine as an SDO device)..

* notice the "Device UUID" shown near the end of the command output, and copy the `/var/sdo/voucher.json` file to your other machine. To do this using the clipboard, run: `cat /var/sdo/voucher.json; echo` and copy the text that is shown (as I do in the accompanying video). Of course, you could instead use `scp` to copy the file from machine to machine.

* once you have saved the `voucher.json` file, you can shutdown the device (e.g., `sudo shutdown -h now`). If this was a real SDO device you would now ship it to the customer together with its `voucher.json` file. **To use the SDO feature, you will need to use this voucher to pre-register the machine with Open Horizon.**

* Perhaps it's worth noting that when you use SDO normally, all of the above is done by the SDO device manufacturer for you. You just get the device and its voucher and you never need to login to the device yourself.

### Pre-Register the Simulated SDO Device (Using its Voucher)

* on any other machine configured with your Open Horizon user credentials. First check your credentials by running `hzn exchange user list`. If that command responds with a JSON structure, you are all set.

* `git clone ...` this repo onto the machine, and `cd` into it.

* save the voucher from the simulated SDO device into this directory using the filename `voucher.json`

* **optionally** (if you wish to customize the example SDO setup provided here) you may edit the `node.policy.json` file to better identify your specific device. In general I recommend providing `properties` that describe the device specifically (e.g., name or serial number), and describe it in terms of its membership in groups of devices (e.g., identifying its specific model, itemizing attached peripherals, and stating what role it will have when deployed, etc.). This is particularly important if you have large numbers of devices, so you can target them using these properties. If you provide a good set of node properties up front, you may never need to change them.

* if you chose to edit the `node.policy.json` to have different property values, then you will most likely need to also modify the `deployment.policy.json` file so the `constraints` in the policy will match the `properties` you have set for the node.

* the example here assumes your simulated SDO device has `amd64` architecture. **If your device has a different hardware architecture then you must change the `arch` field to the appropriate Open Horizon architecture name for your device** (e.g., `arm32` or `arm64`) in the `deployment.policy.json` file.

* **optionally** you may change the service that is referenced in the `deployment.policy.json` file. The example here deploys the `IBM/ibm.helloworld` service. This service is always published automatically in every Open Horizon Management Hub so it is always available for deployment. If you wish to deploy a different service, you must publish that service, and then put its 4-tuple identifier in the `service` section of the `deployment.policy.json` file.

* when you are ready, you can import the device's voucher with the command below:

```
hzn voucher import voucher.json --policy node.policy.json
```

* you can verify the success of that command by running:

```
hzn exchange node list
```

* you will also need to publish your deployment policy with the command below (optionally you may use any name you like instead of "mypolicy"):

```
hzn exchange deployment addpolicy --json-file deployment.policy.json mypolicy
```

* you can verify the success of that command by running:

```
hzn exchange deployment listpolicy
```

### Step Back and Watch the Fireworks!

Now your simulated SDO device can be installed in the field!

Go ahead and power up the simulated SDO device. It will then:

* automatically reach out to the SDO Rendezvous Server you configured (Intel's server by default)

* the SDO Rendezvous Server will coordinate with the software on your simulated SDO device to install the Open Horizon Agent software and configure it with your `node.policy.json`

* when the Agent comes up, and is registered, an AgBot from the Open Horizon Management Hub will start propsing an agreement, using your `deployment.policy.json`

* the Agent will accept, downlooad and verify the service workload, and run it

* within a few minutes your simulated SDO device should be running your specified service



