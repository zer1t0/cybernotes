# Wifi

## Terminology

Here are some terms commonly used in Wifi context with a brief definition:

- **AP** (Access Point): Device that provides wifi, usually a router.

- **WAP** (Wireless Access Point): Same as AP.

- **STA** (Base Station / Station): Device that connects to AP to get access
  to the network. Base station is usually a PC or smartphone.

- **BSS** (Basic Service Set): Most basic infrastructure network type, with just
  one AP.

- **BSSID** (Basic Service Set ID): The identifier of the BSS. Since it is only
  one AP, its MAC address is used as BSSID.

- **ESS** (Extended Service Set): Infrastructure network type that can have more
  than one AP.

- **ESSID** (Extended Service Set ID): The identification of a network that can
  have more than one AP. It is usually a descriptive name like "Coffe Wifi".

- **SSID** (Service Set ID): Same as ESSID.

- **Beacon**: Frame sent by AP to announce the network. It includes the network
  configuration, as its ESSID (or not in case of hidden networks) or its
  channel.

## IEEE 802.11

[IEEE 802.11](https://en.wikipedia.org/wiki/IEEE_802.11) is the wifi protocol family. There are several versions:

| Generation | IEEE standard | Year | Radio frequency band (GHz) |
|------------|---------------|------|----------------------------|
| Wifi 0     | 802.11        | 1997 | 2.4                        |
| Wifi 1     | 802.11b       | 1999 | 2.4                        |
| Wifi 2     | 802.11a       | 1999 | 5                          |
| Wifi 3     | 802.11g       | 2003 | 2.4                        |
| Wifi 4     | 802.11n       | 2009 | 2.4, 5                     |
| Wifi 5     | 802.11ac      | 2013 | 5                          |
| Wifi 6(E)  | 802.11ax      | 2021 | 2.4, 5, (6)                |
| Wifi 7     | 802.11be      | 2024 | 2,4, 5, 6                  |
| Wifi 8     | 802.11bn      |      | 2.4, 5, 6                  |

As we can see the different versions operate can opperate in different radio
frequency bands:
- [2.4 Ghz band](https://en.wikipedia.org/wiki/List_of_WLAN_channels#2.4_GHz_(802.11b/g/n/ax/be)) with 14 channels (From 1 to 14)
- [5 Ghz band](https://en.wikipedia.org/wiki/List_of_WLAN_channels#5_GHz_(802.11a/h/n/ac/ax/be)) with more than 30 non contiguos channels
- [6 Ghz band](https://en.wikipedia.org/wiki/List_of_WLAN_channels#6_GHz_(802.11ax_and_802.11be))

Apart from the physical layer, IEEE 802.11 also defines the logical part of the
protocol. The information is transmitted in packets called **frames**, which are
handled by two sublayers in data link layer in the [OSI model](https://en.wikipedia.org/wiki/OSI_model): **MAC** and
**LLC**.

```

|     ......         |
|--------------------|
| Network layer (IP) |
|--------------------|
| Data Link layer    |
| .----------------. |
| |       LLC      | |
| |----------------| |
| |       MAC      | |
| '----------------' |
|--------------------|
|   Physical layer   |
'--------------------'

```


- **MAC** (Media Access Control): MAC is the lower layer, closer to the
  physical layer. A MAC frame is called called MPDU (MAC Protocol Data Unit) but we
  will call it frame for clarity, and it is composed by three parts:
  + Header: Indicates the 802.11 frame type and some other flags.
  + Body: It contains the payload of the frame, whose format varies depending on
    the type of the frame. In case of being a data (or QoS data) frame type, it
    contains the LLC layer and upper layers data, this kind of body is called a
    MSDU (MAC Service Data Unit).
  + Trail: Also known as FCS (Frame Check Sequence), the trail is used as CRC
    error detection over the frame.

- **LLC** (Logical Link Control): LLC is the upper layer, used between the
  MAC layer and the network layer to provide information about what kind of data
  is being transported in an upper layer. It is only included in the data or Qos
  data frames, since those are the frames defined to carry upper protocols data.

Here is a diagram that shows how the two layers interoperate:
```

+------------+-----------------------+-------------+
| MAC header |          Body         | Trail / FCS |
+------------+-----------------------+-------------+
             |                       |
             +------------+----------+
             | LLC Header | LLC Body |
             +------------+----------+
```


The 802.11 specification defines several types and subtypes of frames, that are
identified by the three first header fields in the MAC header:

```
MAC Header: 32 bytes
+------------------------------------------------------------------------+
| Frame Control (2) | Duration / ID (2) | Address 1 (6) | Address 2 (6)  |
|------------------------------------------------------------------------|
| Address 3 (6) | Sequence Control (2) | Address 4 (6) | Qos Control (2) |
+------------------------------------------------------------------------+
```

- Frame Control: Flags that describe the packet. Explained below.
- Address 1, 2, 3 and 4: MAC addresses whose meaning depends on the *To DS* and
  *From DS* flags of Frame Control.

```
Frame Control: 16 bits (2 bytes)

+-------------------------------------------------------------------------+
| Protocol Version (2) | Type (2) | Subtype (4) | To DS (1) | From DS (1) |
|-------------------------------------------------------------------------|
|  More frags (1)   |  Retry (1)   |  Power Mgmt (1)   |  More Data (1)   |
|-------------------------------------------------------------------------+
| Proctected Frame (1) | Order(1) |
+---------------------------------+
```

- **Version**: Two bytes that are always 0.
- **Type**: Two bits that allows to define the following types of packets:
  + 00: Management
  + 01: Control
  + 10: Data
  + 11: Reserved

- **Subtype**: Four bits that define the subtype of the packet and have different
  meaning depending on the type.


Here are some of the more important message subtypes from security perspective
([Wikipedia provides the whole table of types](https://en.wikipedia.org/wiki/802.11_frame_types)):

Management frames (00):
- **0000 - Association request**: Sent by a station to start the association
  with an AP. The station indicates its supported characteristics.
- **0001 - Association response**: Sent by an AP as response to a station
  Association request, it includes a code to indicate if the association was
  successful and the characteristics of the connection.
- **0010 - Reassociation request**
- **0011 - Reassociation response**

- **0100 - Probe request**: It is sent by the station to request the APs to
  announce themselves. It is usually a broadcast message that can be directed to
  all APs, or just those with an specific SSID.

- **0101 - Probe response**: It is sent by the AP to the station as answer to the
  probe request and announce its characteristics. It is similar to a Beacon
  frame.

- **1000 - Beacon**: A broadcast frame sent by the AP to announce its presence and
  its characteristics. It is similar to a Probe response frame.

- **1010 - Disassociation**: Sent by the AP to the station to indicate that it has
  been disassociated. The   bad thing is that this frame is not checked to
  really come from the AP, as anyone can fake the source MAC.

- **1011 - Authentication**: An authentication packet sent by an station and AP
  before the association. It has the same structure for request and response. It
  was intended for authentication, but nowadays is just a formality, since it is
  completely useless.

- **1100 - Deauthentication**: Sent by the AP to the station to indicate that it
  has been unauthenticated. Same as the disassociation frame, it can be faked by
  anyone by setting the AP MAC, and allowing that station to require to
  reauthenticate again.

Control frames (01):
- **1101 - ACK**: Sent to acknowledge a frame reception.

Data frames (10):
- **0000 - Data**: Used by both the stations and APs to send data of inner
  layers. WPA handshake and EAPOL messages are sent under these or Qos Data
  frames.
- **1000 - QoS Data**: Same as Data frame, used to send data of inner layers,
  the only difference are that these frames include the QoS flags, that can be
  used. WPA handshake and EAPOL messages are sent under these or Data frames.


> In case you want to filter in Wireshark for the type and subtype, you can use
> the filter `wlan.fc.type_subtype`, that requires you to specify the type and
> subtype together, for example for Beacon you can specify
> `wlan.fc.type_subtype = 0b001000` (or `0x8` or just `8`).
> Here is a list of common packets:
> https://travelingpacket.com/2017/09/26/802-11-wireshark-filters/


- https://howiwifi.com/2020/07/13/802-11-frame-types-and-formats/


## Wifi card management

### Show MAC address

We can get the MAC address of any interface with `ip link show`:
```
$ sudo ip link show wlan0
7: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ieee802.11/radiotap 6e:04:0d:04:f4:ac brd ff:ff:ff:ff:ff:ff permaddr 10:c7:3f:d2:04:e0
```

### Change interface MAC address

We can set manually a MAC address with the `ip link set` command:
```
sudo ip link set wlan0 down
sudo ip link set dev wlan0 address 6e:04:0d:04:f4:ac
sudo ip link set wlan0 up
```

Additionally, we can set a random MAC address with `macchanger`:
```
sudo ip link set wlan0 down
sudo macchanger wlan0 --random
sudo ip link set wlan0 up
```

### Scan available wifis (managed mode)

We can scan for available wifis with `iwlist <iface> scan`:
```
sudo iwlist wlan0 scan | grep ESSID
                    ESSID:"My Wifi"
                    ESSID:"Shop Wifi"
```

Resources:
- 2024 [Connecting to a Wireless Network Through Linux CLI](https://www.baeldung.com/linux/connect-network-cli) by Quincy Dalton


### Connect to WPA network

We can use `wpasupplicant` to connect to a wifi network:
```
sudo apt update
sudo apt install wpasupplicant
```

First we need to create a file with the authentication details:
```
wpa_passphrase <essid> "<password>" | sudo tee /etc/wpa_supplicant.conf
```

Then use the file along with the wifi interface to connect with the network:
```
sudo wpa_supplicant -i <iface> -c /etc/wpa_supplicant.conf
```

Moreover, we can pass the `-B` to run wpa_supplicant in background.

Be aware that WPA supplicant only manages the Wifi layer, so if we want to get
an IP address, we need to perform a DHCP request:
```
sudo dhclient <iface>
```

Resources:
- 2024 [Connecting to a Wireless Network Through Linux CLI](https://www.baeldung.com/linux/connect-network-cli) by Quincy Dalton


### Set monitor mode

To set monitor mode with the `iw` command:
```
sudo ip link set <iface> down
sudo iw <iface> set monitor none
sudo ip link set <iface> up
```

To set monitor mode with aircrack `airmon`:
```
sudo airmon-ng start <iface>
```

Resources:
-  [How to Put WiFi Interface into Monitor Mode in Linux?](https://www.geeksforgeeks.org/linux-unix/how-to-put-wifi-interface-into-monitor-mode-in-linux/)

### Retrieve current wifi interface channel

To know the current channel of our wifi interface `
```
$ sudo iw wlan0 info
Interface wlan0
	ifindex 4
	wdev 0x100000001
	addr d2:3b:fd:88:aa:cc
	type monitor
	wiphy 1
	channel 4 (2427 MHz), width: 20 MHz (no HT), center1: 2427 MHz
	txpower 20.00 dBm
```

We can also use `iwlist`:
```
$ sudo iwlist wlan0 freq | grep Current
          Current Frequency:2.427 GHz (Channel 4)
```

### List wifi interface supported channels/frequencies

We can list the supported frequencies and channels with `iwlist`:
```
$ sudo iwlist wlan0 freq
wlan0  13 channels in total; available frequencies :
          Channel 01 : 2.412 GHz
          Channel 02 : 2.417 GHz
          .....stripped.....
          Channel 12 : 2.467 GHz
          Channel 13 : 2.472 GHz
          Current Frequency:2.412 GHz (Channel 1)
```
In this case the card only supports 2.4Ghz.

We can see here an example of a wifi card that supports both 2.4 and 5 Ghz:
```
$ sudo iwlist wlan0 freq
wlan0  32 channels in total; available frequencies :
          Channel 01 : 2.412 GHz
          Channel 02 : 2.417 GHz
          .....stripped.....
          Channel 132 : 5.66 GHz
          Channel 136 : 5.68 GHz
          Current Frequency:2.427 GHz (Channel 4)
```


### Change wifi car channel

We can change the wifi card channel by using the `iw` utility:
```
sudo iw <iface> set channel <number>
```

We can see the supported and current channel of our wifi card with
`sudo iwlist <iface> freq`.

We may find the following error:
```
$ sudo iw wlan0 set channel 36
kernel reports: Channel is disabled
command failed: Invalid argument (-22)
```
This means that our wifi card doesn't support the given channel, maybe because
we are setting a 5 Ghz channel on a wifi card that only supports 2.4 Ghz.

### Wifi card Troubleshooting

#### Error loading mt76662_rom_patch.bin

For the following error:
```
mt76x2u 2-3:1.0: firmware: failed to load mt76662_rom_patch.bin (-2)
```

A solution in Debian is to install the `firmware-misc-nonfree` package that
contains such firmware:
```
sudo apt install firmware-misc-nonfree
```

## Snif in 5GHz with airodump

We can use the `--band a` option to listen in the 5 Ghz band (which it was first
defined in the 802.11**a** version):
```
sudo airodump-ng --band a wlan0mon
```

Resources:
- [Deauth 5GHz WiFi using mdk4 & aircrack-ng](https://docs.spacehuhn.com/blog/5ghz-deauther/)

## PMKID Attack

```
sudo apt update
sudo apt install hcxtools aircrack-ng
```

Resources:
- [New attack on WPA/WPA2 using PMKID](https://hashcat.net/forum/thread-7717.html)
- [hcxdumptool](https://github.com/ZerBea/hcxdumptool) by ZerBea and [others](https://github.com/ZerBea/hcxdumptool/graphs/contributors)
