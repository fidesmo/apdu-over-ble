# apdu-over-ble

Specification of a Bluetooth Low Energy service and protocol to transmit [APDU](https://en.wikipedia.org/wiki/Smart_card_application_protocol_data_unit) commands and responses. Its purpose is to standardize communication with [Secure Elements](https://www.globalplatform.org/mediaguideSE.asp) installed in devices having BLE connectivity, like smart watches and other wearables.

This repository contains discussion materials and the different versions of [the specification](https://github.com/fidesmo/apdu-over-ble/tree/master/spec).

## Other resources
To allow easy testing of this specification, Fidesmo has open sourced client and server prototypes implemented as Android apps:

### BLE Client
The BLE Client prototype is available in this repository: [apdu-over-ble-android](https://github.com/fidesmo/apdu-over-ble-android)
It needs to be run on an Android device with Bluetooth 4.0 or newer.

### BLE Server
The BLE Server prototype is available in this repository: [android-ble-server](https://github.com/fidesmo/android-ble-server)
It needs to be run on an Android device with 
- Bluetooth 4.0 or newer
- NFC capabilities


To fully test the BLE Client --> BLE Server --> Secure Element connection, it is necessary to have a contactless device, like for example a [Fidesmo Card](https://developer.fidesmo.com/fidesmocard).

