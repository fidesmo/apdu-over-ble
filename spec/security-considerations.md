# Security Considerations

## Introduction

Enabling an always-on wireless connection to a Secure Element which has a range of several dozen meters forces us to reevaluate the threat/countermeasure analysis typically applied to contactless cards. The most important difference is that the contactless interface is very short range, so the user is usually aware that a card-reader interaction is taking place.

This document discusses the different threats that a Secure Element embedded in a device with Bluetooth capabilities may encounter, and suggests some countermeasures. Its purpose is to serve as the starting point of a security analysis that will have to be tailored for the specific device, use cases and utilization scenarios.

## Threats

### Identity Tracking
Bluetooth is able to frequently change private addresses so that third parties cannot associate them to the identity of a user.
But if third parties are able to read a Secure Element’s UID or CardID, which are permanent identifiers, they could associate them to a user. That might happen if the device’s BLE is always on, and the pairing process does not require any interaction on the device side.

Countermeasure: See *User Presence Verification* below.

### Data Confidentiality
Bluetooth LE can encrypt data using a long term key. But an unauthenticated encryption can be broken by a device that listens to the initial ‘bonding’ process (in subsequent connections, master and peripheral will use the long term key they created the first time).

The APDUs exchanged over the BLE link can be encrypted from the server to the Secure Element: in fact, all card management operations use encrypted APDUs. 
On the application level, any applet should encrypt its communication with a server - also because APDUs travel through a mobile phone, which is considered to be an untrusted environment.

Conclusion: eavesdropping does not degrade significantly the system’s security, because authenticated encryption should be applied at APDU level.

### Data Authentication
Same reasoning as in *Data Confidentiality* applies. Communication should be authenticated at APDU level, especially to avoid man-in-the-middle attacks.
Authentication at BLE level adds strong requirements on the BLE device, which might not be feasible.

### User activity impersonation
Some NFC applications rely on the user performing an action: for example, One-Time-Password generators require that the user taps the phone with a contactless card containing the secret seed.
If a BLE peripheral replaces the contactless card, and it is always on, a rogue app on the mobile phone could obtain OTPs without any kind of user activity.

Countermeasure: See *User Presence Verification* below.

### Denial of service
After a few unsuccessful authentication attempts, Secure Elements get into an unusable state to protect the credentials they are carrying. 
Usually a smart card (as a physical object) is much less valuable that the resources it is protecting. But this calculation can be different if the Secure Element is embedded in a high end device such as a smartwatch.

Two possible DoS attacks using this feature:
- An attacker connects to any nearby BLE device. After observing the characteristics, uses the APDU Commands characteristic to send authentication packets to the Secure Element. Since the hacker does not have SE keys, they will fail. After a few such attempts, the SE shuts down.
- A malicious app communicates with a device via BLE. It sends authentication APDUs that will fail repeatedly (the malicious app does not have the keys), until the device's SE goes into an unusable state as self-protection.

Countermeasures: *User Presence Verification* would limit the chances of both attacks happening. *Secured pairing* protects against the first attack but not against the second, because all applications installed on the client device (for example a smartphone) can access paired devices.

## Countermeasures

### User presence verification
The device's BLE capabilities are only active after user intervention (e.g by pressing a button, BLE is active for 15 min). Therefore, the device's Secure Element will only be reachable when the user is ready to operate with it.

### Secured pairing
Include some parameter in the pairing procedure so that it can only be performed by the legitimate user: either displaying a code on the device that has to be input on the Bluetooth client, or distributing such code with the device's materials.

