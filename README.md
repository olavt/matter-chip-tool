# Using the Matter chip-tool
### Author: [Olav Tollefsen](https://www.linkedin.com/in/olavtollefsen/)

## Introduction

This article containes example of the most useful Matter chip-tool commands. It also goes threough some scenarios for working
with Matter devices across different ecosystems like Apple Home.

## Pairing commands

The pairing commands are used to commission device into a Matter Fabric. A device can join multiple fabrics in order
to be admistrated from different ecosystems (Apple Home, Google Home, Amazon Alexa and others)

### Commission a Matter over Thread Device using the chip-tool

```
./chip-tool pairing ble-thread 5535 hex:0e080000000000010000000300001435060004001fffe00208134a64c3f10ccb930708fd25da7f3d5c8ff2051019347339f0b597887b0f6b5f1bed98d3030f4f70656e5468726561642d353032310102502104105ebd52c120a93892c7ab2a42dc6fe8d40c0402a0f7f8 20202021 3840
```

### Commission a Matter over WiFi Device using the chip-tool

If you have a QR-code for the device it needs to be decoded to use it with the chip-tool.

Use a tool to decode the scanned QR-code into a code like this:

MT:6FCJ142C00KA0648G00

Use the chip-tool to decode the QR-code information:

```
$ ./chip-tool payload parse-setup-payload MT:6FCJ142C00KA0648G00
```

Commission the device:

```
$ ./chip-tool pairing ble-wifi <node_id> <SSID> <password> <setup_PIN_code> <discriminator> --bypass-attestation-verifier true
```

### Multi-admin

These commands are used to allow devices that are already part of a fabric to be paired to another fabric. The device can then be administrated from controllers in different ecosystems.

### Prepare a device for pairing to another fabric

Open the commissioning window on the paired Matter device by using the following command pattern:

```
$ ./chip-tool pairing open-commissioning-window <node_id> <option> <window_timeout> <iteration> <discriminator>
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
$ ./chip-tool pairing open-commissioning-window 5535 1 600 1000 3840
```

### Forgetting the already-commissioned device

```
$ ./chip-tool pairing unpair <node_id>
```

## Device commands

### Turn light On

```
.chip-tool onoff off 5535 1
```

### Read basic information from the Matter device

Every Matter device supports the Basic Information cluster, which maintains the collection of attributes that a controller can obtain from a device. These attributes can include the vendor name, the product name, or the software version.

Use the CHIP Tool's read command on the basicinformation cluster to read those values from the device:

```
$ ./chip-tool basicinformation read vendor-name <node_id> <endpoint_id>
$ ./chip-tool basicinformation read product-name <node_id> <endpoint_id>
$ ./chip-tool basicinformation read software-version <node_id> <endpoint_id>
```

You can also use the following command to list all available commands for the Basic Information cluster:

```
$ ./chip-tool basicinformation
```

