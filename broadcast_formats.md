# Official Ruuvi Broadcast formats

RuuviTags are Bluetooth LE sensor beacons. While Bluetooth SIG has defined standard ways
to represent sensor data via GATT connection, there is no widely adopted standard for
broadcasting the sensor information in one-to-many configuration. Therefor a custom representation of the data is required.

By default RuuviTags broadcast the sensor data in advertisement packet as a manufacturer
specific data. Some of the first versions of RuuviTags had the data encoded into 
an Eddystone URL, however since the Google has discontinued their Nearby notfications
the URL dataformats are depreciated.

Advantages of broadcasting sensor data are simplicity, as there is no connection negotiation
required and density, as a single receiver can listen to hundreds of the sensors.

Disadvantages are that there is no guarantee or acknowledgement that a message is received
and there is no protection against spoofing the data or Man-In-The-Middle attacks.
A sequence number allows to receiver to detect recent duplicate as well as missing packets. 
Format 8 uses encryption to mitigate hacking issues.

The data formats are identified by the first byte of the advertisement payload. A typical
RuuviTag Bluetooth broadcast will contain: 
 - Preamble
 - Access Address
 - Advertising Channel Protocol Data Unit
   * MAC address of the tag
   * Flags of the advertisement
   * Ruuvi data
 - Cyclic redundancy checksum

For a detailed explanation on Bluetooth advertisement packets please refer to Bluetooth 
SIG [post](https://www.bluetooth.com/blog/bluetooth-low-energy-it-starts-with-advertising/).

Most scanners will return only Advertising channel PDU to the application. 
<!--  ++++ should this example be ommitted if favor of the docs for format 5 ?   DGG 10/21/19 +++++++++++ -->
An example format 5 advertised
value could be `0x0201061BFF99040505941A5BC7B1FFE0001C043867366F2497ED4DFAE75678`. This is
interpreted as:
 - `0x020106`
   * `0x02`: 1 byte length + 2 bytes payload for total of 3 bytes
   * `0x01`: Type Flags. 
   * `0x06`: Flag value: LE General Discoverable Mode, BR/EDR not supported
 - `0x1BFF99040505941A5BC7B1FFE0001C043867366F2497ED4DFAE75678`
   * `0x1B`: 1 byte length + 27 bytes payload for a total of 28 bytes
   * `0xFF`: Type Manufacturer Specific data
   * `9904`: Manufacturer: Ruuvi Innovations, Least Significant Byte first.
   * `0x05`: Header byte of Ruuvi data format, 0x05. 
   * `5941A5BC7B1FFE0001C043867366F2497ED4DFAE75678`: Data payload, described under section Data Format 5. 

It should be noted that Ruuvi data formats can be used in many different transport layers.
It is allowed to send other data alongside same advertisement packet using BLE5 extended
advertising or interleaving the advertisements with other data formats. Therefore parsers
cannot identify Ruuvi data by length of the BLE advertisement or blacklist devices sending
different data. The advertisements may be connectable or non-connectable. 

Some formats include the MAC address which is needed for applications running on Apple products.

The correct way to parse the data is to identify if there is a manufacturer specific data field
from Ruuvi Innovations and then check if the header of the data payload matches Ruuvi data formats.

Ruuvi tags allow the user to select from multiple formats included in the firmware.

# Data format 1 
**Lifecycle: Obsolete.**

Data format 1 used base91 encoding to pack data into URL parameter. However it wasn't
compatible with many browsers and was therefore discontinued

# Data format 2
**Lifecycle: Obsolete.**

Data format 2 was used on Kickstarter devices. It is described in detail [here](./dataformat_04.md).
The data is encoded on Eddystone URL with a URL-safe base64 encoding and interpreted via server-side javascript
of the website of the URL.

# Data format 3 (also known as RAWv1)
**Lifecycle: In production.**

RAWv1 is a primary data format in 1.x and 2.x branch firmware. It includes a 1 byte humidity field and 2 byte temperature, atmospheric pressure, X/Y/Z accelerometer and battery fields.  RAWv1 has been included in tags shipped since June 2018 . 
Described in detail [here](./dataformat_03.md).

# Data format 4
**Lifecycle: Depreciated.**

The data is encoded on Eddystone URL with a URL-safe base64 encoding and interpreted via server-side javascript
of the website of the URL.

This data format will be removed in 2.x branch firmware. 
Format 4 was the primary data format in RuuviTags shipped before June 2018. It is described in detail [here](./dataformat_04.md).

# Data format 5 (also known as RAWv2)
**Lifecycle: In production.**

RAWv2 is a primary data format in 2.x branch firmware. Similar to format 3 it extends humidity to 2 bytes and includes fields for TX power, a movement counter, a sequence number and the MAC address.
It has been available for update since January 2019 and is included in tags shipped since October 2019. Format is described in detail [here](./dataformat_05.md).

# Data format 8 (Encrypted environmental data)
**Lifecycle: Proposal.**

Encrypted data format will be based on manufacturer specific data. Based on format 5, the movement counter is extended to 2 bytes and the sequence number to 3 bytes. The accelerometer fields may be dropped. It will be implemented in 3.X firmware once specification is agreed on. Proposal is described in detail [here](./dataformat_08.md).

# Unofficial Ruuvi data formats
**Lifecycle: In production**

If you wish to use your own data formats on RuuviTags, your are allowed to use 
Ruuvi Innovations Bluetooth SIG ID 0x0499 under following conditions:
 - The hardware is manufactured by Ruuvi Innovations
 - The first byte of data payload (the format) is in range of 0xF0 ... 0xFF
