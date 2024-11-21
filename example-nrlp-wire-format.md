# Examples of Wire Format Options

## Reminder

~~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| OPTION_V4_NRLP|     Length    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   NRLP Instance Data Length   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|U U P E|U U R R D S|    TC     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Committed Information Rate   |
|              (CIR)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Committed Burst Size (CBS)   |
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~


## One Throughput Instance

Let's consider the example of this advice:

~~~
{
    "throughput-advice": [
        {
            "direction": 0,
            "scope": 0,
            "tc": 0,
            "cir": 50,
            "cbs": 10000
        }
    ]
}
~~~

* direction ==> D-flag
* scope ==> S-flag

### DHCP

This corresponds to the following NRLP DHCP option:

~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    0xE0       |     0x0C      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x0A      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x32      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x27      |     0x10      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

* OPTION_V4_NRLP: 0xE0 (224)

#### ISC DHCP Server

To add NRLP option to [dhcp.conf](https://kb.isc.org/docs/isc-dhcp-44-manual-pages-dhcp-options):

~~~
option nrlp code 224 = string;
option nrlp 00:0A:00:00:00:00:00:32:00:00:27:10;
~~~

#### Kea DHCP Server

To add NRLP option to [kea-dhcp4.conf](https://kea.readthedocs.io/en/kea-2.2.0/arm/dhcp4-srv.html#dhcpv4-private-options):

~~~
"Dhcp4": {
    ...
    "option-def": [
        {
            "name": "nrlp",
            "code": 224,
            "type": "binary",
            "space": "dhcp4"
        },
        ...
    ],
    "option-data": [
        {
            "name": "nrlp",
            "space": "dhcp4",
            "csv-format": false,
            "data": "00:0A:00:00:00:00:00:32:00:00:27:10"
        },
        ...
    ],
    ...
}
~~~

#### Client Side

Add the following to `dhclient.conf`:

~~~~
option nrlp code 224 = string;
also request nrlp;
~~~~

## One Throughput Instance Per Direction

Let's consider the example of this advice:

~~~
{
    "throughput-advice": [
        {
            "direction": 0,
            "scope": false,
            "tc": 0,
            "cir": 50,
            "cbs": 10000
        },
        {
            "direction": 1,
            "scope": false,
            "tc": 0,
            "cir": 40,
            "cbs": 8000
        },
    ]
}
~~~

### DHCP

This corresponds to the following NRLP DHCP option:

~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    0xE0       |     0x18      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x0A      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x32      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x27      |     0x10      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x0A      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x80      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x28      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x00      |     0x00      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     0x1F      |     0x40      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~

* OPTION_V4_NRLP: 0xE0 (224)

#### ISC DHCP Server

To add NRLP option to [dhcp.conf](https://kb.isc.org/docs/isc-dhcp-44-manual-pages-dhcp-options):

~~~
option nrlp code 224 = string;
option nrlp 00:18:00:00:00:00:00:32:00:00:27:10:00:0A:00:80:00:00:00:28:00:00:1F:40;
~~~

#### Kea DHCP Server

To add NRLP option to [kea-dhcp4.conf](https://kea.readthedocs.io/en/kea-2.2.0/arm/dhcp4-srv.html#dhcpv4-private-options):

~~~
"Dhcp4": {
    ...
    "option-def": [
        {
            "name": "nrlp",
            "code": 224,
            "type": "binary",
            "space": "dhcp4"
        },
        ...
    ],
    "option-data": [
        {
            "name": "nrlp",
            "space": "dhcp4",
            "csv-format": false,
            "data": "00:18:00:00:00:00:00:32:00:00:27:10:00:0A:00:80:00:00:00:28:00:00:1F:40"
        },
        ...
    ],
    ...
}
~~~

