# Please Send these to me
  - jonezy7173@gmail.com
  - jjones@ncdoc.navy.mil
  - 419-560-3836


# NSM Engineer Course

### Order for Test
  1. Installing
  2. Networking
  3. Interface Optimization
  4. Test Collection


# pfSense Installation

## Hardware Setup
    1. ESC - enter BIOS Setup
    2. F11 - boot new

    - this is a fast setting, be ready!!

## Initial Installation
    1. A - accept
    2. I - install (OK)
    3. <enter> - default US Keymap
    4. Auto - Guided Disk Setup
    5. Accept All Defaults
    <br> ... install runs ...

## Initial Config

1) Assign Interfaces

```
1 - assign interface ,
n - skip vlan Setup
en0 - WAN Interface
en1 - LAN Interface
(nothing to finish) Enter
y - proceed

reloading....

```
2) Set interface IPs

```
2 - set WAN interface
1 - WAN Interface
y - dhcp
n - dhcp6
blank for "none"
y - http webconfig protocol

2 - LAN interface
172.16.10.1/24
<Enter> for none
<Enter> for none - disable ipv6
y - enable dhcp
172.16.10.100 - range start
172.16.10.254 - range end
y - http webconfig protocol

```

## Finishing Up (Wizard)

#### In order to access the router, complete the following: <br>
  1. Set your lab machine to static ip 172.16.xx.10
  2. plug into the LAN interface (LAN2 on front face) and browse to 172.16.xx.1
  3. complete first login to web Guided

default creds:
```
admin / pfsense
```

### Wizerd > Pfsense Setup > General Information

```
set hostname sg10-pfsense localdomain == local.lan
primary dns
uncheck block private networks
LAN ip address - 172.16.10.1
subnet mask - 24
set (and document) new admin password - New password is: training

Add a WAN rule to allow laptops to connect to sensors

  - Source: 192.168.2.0/24 any any any

Disable DNS resolver

Enable DNS forwarder

Add a custom option under DNS Forwarder to allow RFC 1918 rebinding of lab-repo

rebind-domain-ok=/lab-repo/
```

# Day 2

## Installing & Configuring CentOS


  1. Plug in CentOS USB
  2. Click on Network & hostname
  3. Change Hostname to sg##.lab.lan
  4. Click "Apply"
  5. Configure top Interface
    - Click on IPv4
    - Click "Method"
    - Set to auto dhcp
  6. Click "Done"
  7. Click ipv6
    - Click "Ignore"
  8. Click "Save" & "Done"

  9. Go to KDUMP
    - Disable KDUMP
    - Click "Done"

  10. Click "Installation Destination"
    - Check both of the disks
    - Click "Done"
    - Reclaim Space
    - Delete all
    - Reclaim Space
    - Click "I will configure partitioning"
    - Click "Done"
    - Click here to create them automatically

    11. Configuring Storage
      - /home
        - Desired Capacity - 1GiB
      - /
        - Desired Capacity - 1GiB
      - swap
        - Desired Capacity - 1GiB

   ### Add Mount Points
      - Click "+" on the bottom left
      #### Set all to 1GiB
      - /tmp
      - /var
      - /var/log
      - /var/log/audit
      - /var/tmp
      - /data/stenographer
      - /data/Suricata
      - /data/kafka
      - /data/elasticsearch
      - /data/zeek
      - /data/fsf

      - /
        - Volume Group
          - Modify
            - Only select the drive that's ~500G
            - Click "Save"

      - /data/zeek
        - Create New Volume Group
          - Select ~1TB drive
          - Name it CentOS_data
          - Click "Save"
      - ###### Add all /data/* mount points to the volume group CentOS_data that you created above

      #### The above was done to separate the ingress/egress data from the system data. That way if one of the logs fills up it doesn't crash the whole system.

      ### Configure storage space for all of the new mount Points
        - /tmp - 5 GiB
        - /var - 50 GiB
        - /var/log - 50 GiB
        - /var/log/audit - 50 GiB
        - /var/tmp - 10 GiB
        - /home - 50 GiB
        - swap - 10 GiB
        - / - leave blank (do this at the end)
        - /data/stenographer - 500 GiB  
        - /data/Suricata - 25 GiB
        - /data/kafka - 100 GiB
        - /data/elasticsearch - leave blank (do this at the end)
        - /data/zeek - 25 GiB
        - /data/fsf -  10 GiB
        ###### Leaving the storage size blank automatically allocates the remaining storage to that mount point(s)

        - Click "Done" & 'Accept'
        - Click begin Installation  
          - don't create root password
        - User creation
          - Create an admin User
          - Password is: training
          - check the box to "Make the user an admin"
          - Click done

        - Unplug USB when it reboots

## Post Install
  1. check hostname:
    `hostname `
    - if you didn't set your hostname correctly
    `sudo hostnamectl set-hostname  `
    2. Check that ipv6 is disabled:

```vim
sudo vi /etc/sysctl.conf
i - to insert

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

ESC
:wq!
```

```vim
sudo vi /etc/hosts
delete the ipv6 line (::1)
navigate to the line and type dd
:wq!
```

```vim
sudo systemctl restart network
sudo systemctl status network
```

#### If you don't get a dhcp address

```vim
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp5s0

BOOTPROTO = dhcp

ONBOOT = yes

:wq!

sudo systemctl restart network
sudo systemctl status network
```

## Tap & SPAN

 - 1st port into pfSense
 - 2nd port into sensor
 - 5th port also into sensor

## Interface Optimization

#### ethtool

  - utility for modifying a network card. We can turn off any kind of offloading or anything that the network card would normally do so that we can capture the traffic in raw format. Allowing the NIC to interact with the traffic before we receive it is not best practice.
#### [Suricata SEPTun Guide - optimization for processing network data](https://www.github.com/pevma/SEPTun/blob/master/SEPTun.pdf)

#### Setup the capture Interface

```

```

## Day 3

### Changed wallpaper to Kung Fu Panda

### Download repo stuff
  #### SSH into sensors

  ```bash

  sudo -s

  cd /etc/yum.repos.d/
  ls
  mv CentOS-* /tmp/

  ls (should now be empty)

  curl -L -O http://192.168.2.20/share/local.repo

  yum makecache fast

  yum install tcpdump

  tcpdump -i eno1

  curl -L -O http://192.168.2.20/share/interface.sh

  mv interface.sh /tmp/

  chmod 777 interface.sh

  sudo ./interface.sh eno1

  vi /etc/sysconfig/network-scripts/ifcfg-eno1

  sudo vi ifcfg-eno1

    BOOTPROTO=none

    ONBOOT=yes

    set =no to on all IPV6 settings

    :wq!

systemctl restart network

systemctl status network
  ```

## Installing & Configuring stenographer
```bash
sudo -s
yum install stenographer
  y

which stenotype

vi /etc/stenographer/config

  "PacketsDirectory": "/data/stenographer/packets"
  "IndexDirectory": "/data/stenographer/index"
  "Interface": "eno1"

  :wq!

stenokeys.sh stenographer stenographer

which stenographer
  should say /bin/stenographer


mkdir -p /data/stenographer/{packets,index}

cd /data/stenographer
ls (should see index and packets)

cd ..

chown -R stenographer:stenographer stenographer/

systemctl start stenographer
systemctl status stenographer

cd stenographer/packets
ls (should see random string of numbers)


```

## Installing & Configuring Suricata

```bash
yum install suricata

vi /etc/suricata/suricata.yaml
  :set nu
  :56
  default-log-dir: /data/suricata
  ESC

  :580
  - interface: eno1
  ESC
  :wq!

vi /etc/sysconfig/suricata
  OPTIONS="--af-packet=eno1 --user suricata"
  ESC
  :wq!

cat /proc/cpuinfo | egrep -e 'processor|physical id|core id' | xargs -l3

lscpu -e 
```
