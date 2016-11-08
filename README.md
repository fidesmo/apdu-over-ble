# apdu-over-ble

Specification of a Bluetooth Low Energy service and protocol to transmit [APDU](https://en.wikipedia.org/wiki/Smart_card_application_protocol_data_unit) commands and responses. Its purpose is to standardize communication with [Secure Elements](https://www.globalplatform.org/mediaguideSE.asp) installed in devices having BLE connectivity, like smart watches and other wearables.

This repository contains discussion materials and the different versions of [the specification](https://github.com/fidesmo/apdu-over-ble/tree/master/spec).

## Other resources: Android prototype
To allow easy testing of this specification, Fidesmo has open sourced an Android app that can perform the server and client roles. The client sends APDUs over BLE to a server running on an NFC-enabled phone that then forwards the APDU commands to a contactless card and returns its response.

The prototype is available in this repository: [apdu-over-ble-android](https://github.com/fidesmo/apdu-over-ble-android)


