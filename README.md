# easy-sdo-oh

This repo exists to help you try out Secure Device Onboarding with Open Horizon

There is a [YouTube video](coming) you can watch that shows me going through these steps.

### Assumptions:

1. You have a Linux device that you would like to setup as an SDO device (i.e., you wish to simulate the manufacture of an SDO device, like those manufactured by Intel and other vendors).

2. You have **user level** credentials for an Open Horizon Management Hub, and you have `HZN_ORG_ID` and `HZN_EXCHANGE_USER_AUTH` configured appropriately on some machine other than the one where you wish to simulate SDO manufactturing.

### Simulate the SDO Manufacturing Steps On Your Device

* login to the machine you wish to make into an SDO device (via ssh, or on the machine console, etc.)

* either `git clone ...` this repo onto that machine, or simply copy the `SDO_DEVICE_STEPS` file onto this machine.

* run the command `source SDO_DEVICE_STEPS` on this machine

* after it has set things up, it will ask you to run a long command involving `simulate-mfg.sh`. Copy that entire long string then paste it and run it. It will take just a few minutes to run.

* copy the "Device UUID" shown near the end of the command output. Alsco copy the `/var/sdo/voucher.json` file to your other machine. To do this using the clipboard, run: `cat /var/sdo/voucher.json; echo` and copy the text that is shown.

* once you have saved the UUID and `voucher.json`, shutdown the device (e.g., `sudo shutdown -h now`). If this was a real SDO device you would now ship it to the customer together with its `voucher.json` file.

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



