# APDU-Service specification

This document specifies the BLE service that transmits Command-APDUs from a smartphone to a BLE device that also contains a Secure Element.

## Roles
- A smartphone app will implement the *GATT Client* role.
- A BLE + SE device will implement the *GATT Server* role (also a BLE Peripheral)

## Pairing and Security
[bluetooth.org reference](https://developer.bluetooth.org/TechnologyOverview/Pages/LE-Security.aspx)
The device shall enforce Security Mode 1, Level 2 (unauthenticated pairing with encryption) before any messages containing APDUs are exchanged.

TO DO: check BLUETOOTH SPECIFICATION Version 4.2

## Client and Server requirements
Each role must support the following:

### GATT Server role
- Write characteristic value
- Read characteristic value
- Issue notifications

### GATT Client role
- Discover primary services by UUID
- Discover characteristics by UUID
- Read characteristic value
- Write characteristic value
- Receive notifications

## Service parameters

### Service UUID
`8e790d52-bb90-4967-a4a5-3f21aa9e05eb`

### Service Characteristics

| Characteristic name                 | Property        | Length     | UUID                                   |
|-------------------------------------|---------------- |------------|----------------------------------------|
| APDU Commands                       | Write with ACK  | 512 bytes  | `9e73ecae-0701-403a-aefc-a1c5f6d16173` |
| Conversation Finished               | Write with ACK  | < 20 bytes | `a7fb8746-c1dc-47a3-b201-c17411617936` |
| APDU Responses Ready                | Notify with ACK | NA         | `fbbe5e92-afdc-40ea-bada-aaf6f7bb8b01` |
| APDU Responses                      | Read            | 512 bytes  | `954727a7-5907-4377-8b5a-7d70951340a9` |
| Max Memory for APDU processing      | Read            | <20 bytes  | `c2a9e13b-35fa-4c9b-9f59-1829d573689e` |

#### Characteristic: APDU Commands
Used by the Client to transmit a sequence of Command APDUs. The sequence is a serialized stream of APDUs following the structure below:

![APDU Command sequence](fig/command-apdu-sequence.png)

Both the number of APDUs and the length of each APDU are encoded using two bytes, so that the resulting value is:

```value = [byte 0] + [byte 1]*256```

#### Characteristic: Conversation Finished
Used by the Client to notify the Server that the APDU exchange has finished so the Server can power the device down.
The payload transmitted is irrelevant.

### Characteristic: APDU Responses Ready
Notification issued by the Server to signal that all the Command APDUs in the sequence have been processed, so the Response APDUs can be retrieved. The payload transmitted is irrelevant.

### Characteristic: APDU Responses
Used by the Client to retrieve the sequence of Response APDUs corresponding to the processing of a previous sequence of Command APDUs. The Client will read this characteristic only after receiving *APDU Responses Ready* notification. Response APDUs will be encoded in the same structure as command APDUs in the figure above.

### Characteristic: Max Memory for APDU processing
Memory, in kilobytes, that the Server can use to store APDU Commands. This limits the number of APDU Commands that can be sent by writing to the *APDU Commands* characteristic.
This characteristic is optional. If missing, the Server indicates that it does not have any memory limitations, so the Client does not need to limit the number of Command APDUs sent in a sequence.

## Packet and payload structure
APDU Commands and APDU Responses data may not fit into a single BLE packet, so they will have to be fragmented by the issuer and then reconstructed by the receiver. The APDU-Service will use the following packet structure:

![BLE packet structure](fig/ble-packet-structure.png)

which contains the following fields:
- `len`: total sequence length
- `totn_pkt`: total packets number
- `pkt_nbr`: sequence number of the fragment, encoded in one byte
- `data`: payload. Fragment of the APDU Commands/Responses sequence

The data follows a big-endian byte order (byte 0 sent first), and little endian bit order in each byte:

![Endianess](fig/endianess.png)


*Length* does not need to be indicated explicitly: packet length is indicated by the lower layers of the BLE protocol. Length will always be the maximum size allowed by the characteristic (MTU-1), except for the last packet. In case the last packet fits exactly in MTU-1, an empty packet must be sent. Since the end of the APDU sequence is easily detected, it is not necessary to indicate the total number of fragments for a given APDU sequence.

## Packet ordering and retry policy

### Commands

When sending a sequence of Command APDUs, the Client sends the different fragments in order, waiting for the ACK from the Server (sent by lower layers of the BLE stack). If the ACK for a packet is not received, that packet will be resent. The `pkt_nbr` can be used on the other side to detect repeated packets (e.g. in case an ACK had been lost).

### Responses

When the Server has finished processing the Command APDUs, it issues an `APDU Responses Ready` notification. Then the Client will fetch the first fragment of the sequence of Response APDUs by reading the `APDU Responses` characteristic.
The `APDU Responses` characteristic will also work as implicit ACK of the previous message. Therefore,
if the Server does not receive an `APDU Responses` read, it will resend either the `APDU Responses Ready` notification or the last fragment of the Response APDUs sequence.


## Sequence diagram
Example of an exchange of Command and Response APDUs:

![Example sequence](fig/example-sequence.png)
