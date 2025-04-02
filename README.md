# Using the Matter chip-tool
### Author: [Olav Tollefsen](https://www.linkedin.com/in/olavtollefsen/)

## Introduction

This article containes example of the most useful Matter chip-tool commands. It also goes threough some scenarios for working
with Matter devices across different ecosystems like Apple Home.

## Build Matter

The following steps are documented here: https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/BUILDING.md

### Checking out the Matter code

```
git clone --recurse-submodules https://github.com/project-chip/connectedhomeip.git
```

### Updating Matter code

```
cd connectedhomeip/
git pull
git submodule update --init

```

### Installing prerequisites on Raspberry Pi 4

#### Installing prerequisites on Linux

```
sudo apt-get install git gcc g++ pkg-config libssl-dev libdbus-1-dev \
     libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev \
     python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev
```

#### Install some Raspberry Pi specific dependencies:

```
sudo apt-get install pi-bluetooth avahi-utils
```

### Configuring wpa_supplicant for storing permanent changes

By default, wpa_supplicant is not allowed to update (overwrite) configuration. If you want the Matter application to be able to store the configuration changes permanently, you need to make the following changes:

1. Edit the dbus-fi.w1.wpa_supplicant1.service file to use configuration file instead by running the following command:

```
sudo nano /etc/systemd/system/dbus-fi.w1.wpa_supplicant1.service
```

2. Change the wpa_supplicant start parameters to::

```
ExecStart=/sbin/wpa_supplicant -u -s -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

3. Add the wpa-supplicant configuration file by running the following command:

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

4. Add the following content to the wpa-supplicant file:

```
ctrl_interface=DIR=/run/wpa_supplicant
update_config=1
```

5. Reboot your Raspberry Pi.

### Prepare for building

Before running any other build command, the scripts/activate.sh environment setup script should be sourced at the top level. This script takes care of downloading GN, ninja, and setting up a Python environment with libraries used to build and test.

```
cd connectedhomeip
```

```
source scripts/activate.sh
```

If this script says the environment is out of date, it can be updated by running:

```
source scripts/bootstrap.sh
```

The scripts/bootstrap.sh script re-creates the environment from scratch, which is expensive, so avoid running it unless the environment is out of date.

### Building the CHIP Tool

```
mkdir out
 ./scripts/examples/gn_build_example.sh examples/chip-tool out
```

## Pairing commands

The pairing commands are used to commission device into a Matter Fabric. A device can join multiple fabrics in order
to be admistrated from different ecosystems (Apple Home, Google Home, Amazon Alexa and others)

### Commission a Matter over Thread Device using the chip-tool

```
./chip-tool pairing ble-thread 5535 hex:0e080000000000010000000300001435060004001fffe00208134a64c3f10ccb930708fd25da7f3d5c8ff2051019347339f0b597887b0f6b5f1bed98d3030f4f70656e5468726561642d353032310102502104105ebd52c120a93892c7ab2a42dc6fe8d40c0402a0f7f8 20202021 3840
```

### Commission a Matter over WiFi Device using the chip-tool

If you have a QR-code for the device it can be decoded to use it with the chip-tool.

Use a tool to decode the scanned QR-code into a code like this:

MT:6FCJ142C00KA0648G00

Use the chip-tool to decode the QR-code information:

```
./chip-tool payload parse-setup-payload MT:6FCJ142C00KA0648G00
```

Commission the device:

```
./chip-tool pairing ble-wifi <node_id> <SSID> <password> <setup_PIN_code> <discriminator> --bypass-attestation-verifier true
```

```
./chip-tool pairing code-wifi <NodeId> <SSID> <password> <short manual pairing code> --bypass-attestation-verifier true
```

```
./chip-tool pairing code <NodeId> <PairingCode>
```

### Multi-admin

These commands are used to allow devices that are already part of a fabric to be paired to another fabric. The device can then be administrated from controllers in different ecosystems.

### Prepare a device for pairing to another fabric

Open the commissioning window on the paired Matter device by using the following command pattern:

```
./chip-tool pairing open-commissioning-window <node_id> <option> <window_timeout> <iteration> <discriminator>
```

Now you can use this pairing command to commission the device:

```
./chip-tool pairing code <NodeId> <PairingCode>
```

In this command:

- \<node_id\> is the ID of the node that should open commissioning window.

- \<option\> is equal to 1 for Enhanced Commissioning Method and 0 for Basic Commissioning Method.

- \<window_timeout\> is time in seconds, before the commissioning window closes.

- \<iteration\> is number of PBKDF iterations to use to derive the PAKE verifier.

- \<discriminator\> is device specific discriminator determined during commissioning.

Note: The \<iteration\> and \<discriminator\> values are ignored if the \<option\> is set to 0.

Example command:

```
./chip-tool pairing open-commissioning-window 5535 1 600 1000 3840
```

### Forgetting the already-commissioned device

```
./chip-tool pairing unpair <node_id>
```

## Device commands

### Turn light On

```
.chip-tool onoff off 5535 1
```

### Change color of a RGB light bulb

```
./chip-tool colorcontrol move-to-color 24939 24701 10 0 0 1 1
```

### Use wildcards

The following wildcards are available:

- For all attributes: 0xFFFFFFFF
- For all clusters: 0xFFFFFFFF
- For all endpoints: 0xFFFF

```
./chip-tool colorcontrol read-by-id 0xFFFFFFFF 1 1
```

### Read basic information from the Matter device

Every Matter device supports the Basic Information cluster, which maintains the collection of attributes that a controller can obtain from a device. These attributes can include the vendor name, the product name, or the software version.

Use the CHIP Tool's read command on the basicinformation cluster to read those values from the device:

```
./chip-tool basicinformation read vendor-name <node_id> <endpoint_id>
./chip-tool basicinformation read product-name <node_id> <endpoint_id>
./chip-tool basicinformation read software-version <node_id> <endpoint_id>
```

You can also use the following command to list all available commands for the Basic Information cluster:

```
./chip-tool basicinformation
```

## Generate Matter Onboarding Codes (QR Code and Manual Pairing Code)

Note! The values used below are defined in "CHIPProjectConfig.h" found in the include folder of your project.

```
// Generate the QR Code
./chip-tool payload generate-qrcode \
  --discriminator 3840 \
  --setup-pin-code 20202021 \
  --vendor-id 0xFFF1 \
  --product-id 0x8004 \
  --version 0 \
  --commissioning-mode 0 \
  --rendezvous 2
```

```
// Generates the short manual pairing code (11-digit).
./chip-tool payload generate-manualcode \
  --discriminator 3840 \
  --setup-pin-code 20202021 \
  --version 0 \
  --commissioning-mode 0
```

```
// To generate a long manual pairing code (21-digit) that includes both the vendor ID and product ID,
// --commissioning-mode parameter must be set to either 1 or 2, indicating a non-standard commissioning flow.
./chip-tool payload generate-manualcode \
  --discriminator 3840 \
  --setup-pin-code 20202021 \
  --vendor-id 0xFFF1 \
  --product-id 0x8004 \
  --version 0 \
  --commissioning-mode 1
```

