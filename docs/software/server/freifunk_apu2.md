# Freifunk node based on APU2

## Schematic network overview

+----------+ +---------+
|          | |         |
| Internet | | Internet|
|  enp0s1  | |   LTE   |
|          | |         |
+----------+ +---------+
      |           |
+----------------------+
|                      |
|        OpenWRT       |
|          VM          |
|                      |
+----------------------+
            |         |
            +---------|--------------------------------+
                      |                                |
                   +----------+                  +------------+
                   |          |                  |            |
                   |    br0   |                  |    br2     |
           +-------| internet |--------+         | management |
           |       |          |        |         |            |
           |       +----------+        |         +------------+
           |             |             |               | |
     +----------+  +----------+  +------------+        | |
     |          |  |          |  |            |        | |
     | Freifunk |  | Freifunk |  |   UniFi    |        | |
     |  Stable  |  |   Beta   |  | Controller |        | |
     |    VM    |  |    VM    |  |  Container |        | |
     |          |  |          |  |            |        | |
     +----------+  +----------+  +------------+        | |
           |             |             |               | |
     +----------------------------------------+        | |
     |          |  |          |  |            |        | |
     | Freifunk |  | Freifunk |  |    UniFi   |        | |
     |  Stable  |  |   Beta   |  |    Mgmt    |        | |
     |  client  |  |  client  |  |            |        | |
     |    br1   |  |    br1   |  |     br1    |        | |
     | vlan 100 |  | vlan 101 |  |   NO vlan  |        | |
     |          |  |          |  |            |        | |
     +----------------------------------------+        | +----------+
           |             |             |               |            |
           |        +--------+         |         +------------+  +------+
           |        |        |         |         |            |  |      |
           |        | Client |         |         | Management |  | APU2 |
           +--------| Access |---------+         |   enp0s3   |  | Mgmt |
                    | enp0s2 |                   |            |  |      |
                    |        |                   +------------+  +------+
                    +--------+
                         |
                    +--------+
                    |        |
                    |   AP   |
                    |        |
                    +--------+
                     |      |
             +----------+ +---------------+
             |          | |               |
             |   SSID   | |     SSID      |
             | freifunk | | freifunk-next |
             |          | |               |
             | vlan 100 | |   vlan 101    |
             |          | |               |
             +----------+ +---------------+

## Install freifunk VMs

1. Download freifunk x86 image from [firmware.freifunk-bs.de](https://firmware.freifunk-bs.de/#stable)
    a. Optionally download parker beta [w.freifunk-bs.de](http://w.freifunk-bs.de/)
2. Unzip the image `gunzip <image>`
3. Copy image to `/var/lib/libvirt/images`
4. Open virtual machine manager
5. open settings
6. Setup a new `brige` via `Network interfaces` and add `enp2s0` as interface, name the bridge `freifunk`
7. Setup new VM with freifunk image, 1st NIC is the `freifunk` bridge, 2nd NIC is the internet providing network. e.g. NAT from $Network.