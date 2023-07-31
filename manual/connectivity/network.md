# Network setting

The hostname is the name of the machine. If no value is specified, the default hostname used is 'mikopbx.local'.

<figure><img src="../../.gitbook/assets/1 (14).png" alt=""><figcaption></figcaption></figure>

## Network interfaces

There are two ways to configure the IP address:

1. DHCP (Dynamic Host Configuration Protocol) can be used for automatic IP address configuration. Enable the 'Use DHCP to obtain network settings' switch. This is recommended for most users. To not rely on DHCP server settings (to provide a specific address), you can disable the switch.
2. If you do not want to use settings obtained from a DHCP server, you can configure the network manually. This requires some knowledge about the network topology. To the right of the IP address, there is a field for Subnet Mask in CIDR format. You should use the alternative format: /8 corresponds to the subnet mask 255.0.0.0, /16 corresponds to 255.255.0.0, and /24 corresponds to 255.255.255.0.

'VLAN ID' - MikoPBX supports virtual network interfaces. This is relevant only for physical PCs. Sometimes a PC may have only one network interface, and it may not be possible to connect a second one physically. Using VLAN, you can create a virtual interface that works 'on top' of the physical one. One of the advantages of using VLAN is that all phone calls can be routed through it, while the network equipment can 'tag' all VLAN traffic and guarantee a stable connection.

The number of network interfaces in MikoPBX is not limited.

<figure><img src="../../.gitbook/assets/2 (13).png" alt=""><figcaption></figcaption></figure>

## **Network topology**

The '**Network interface with internet access**' is the primary network interface through which access to external addresses (non-local) will be established.

If no **DNS server** address is specified, the default server 8.8.8.8 will be used.

Depending on your network topology, you need to perform the following steps to configure MikoPBX. The PBX can be behind a network router, which is the most common scenario, or it can have a public IP.

* If the PBX is behind a **router**, you need to check the '**This station is located behind a NAT router' option.**
* If you know the **external address** of the station (IP or domain name) and **have forwarded the ports** of the PBX to the external world, it is recommended to fill in the fields '**External IP address of your router**' or '**External hostname of your router**'.

For all addresses that are not local to the PBX, the station will be represented by the external address:

If 'External IP address of your router' is empty and 'External hostname of your router' is filled, the PBX will be represented by the hostname (External hostname) field.

{% hint style="info" %}
The external IP address is mandatory to fill in. If a domain name is specified, it takes priority, and the external IP address field is not used.
{% endhint %}

<figure><img src="../../.gitbook/assets/3 (26).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
When enabling the option 'This station is located behind a NAT router,' it is mandatory to specify the external address or hostname of the router. Additionally, you need to perform port forwarding on the router for SIP port 5060 and RTP ports 10000-10200 to the local address of the PBX.

If your provider allows registration and you do not need to connect external subscribers, you can choose not to enable the option 'This station is located behind a NAT router,' even if the PBX is behind a NAT router.
{% endhint %}

## Manual configuration of network routes

1. Go to the '**System**' → '**System file customization**' section.

<figure><img src="../../.gitbook/assets/4 (3).png" alt=""><figcaption></figcaption></figure>

2. Open the file **'/etc/static-routes**' for editing.

<figure><img src="../../.gitbook/assets/5 (24).png" alt=""><figcaption></figcaption></figure>

3. Select the '**To replace all**' mode and insert the rule. For example, _'route add -net 54.246.198.136 netmask 255.255.255.255 gw 172.16.32.15 dev eth0_'

<figure><img src="../../.gitbook/assets/31-05-2023 0-59-21.png" alt=""><figcaption></figcaption></figure>

We specify to the operating system that the specified IP address **54.246.198.136** can be found through the network interface 'eth0' and the request should be directed to the gateway (**172.16.32.15**).

The netmask '**255.255.255.255**' indicates that the rule will only be applicable to the address **54.246.198.136**. If you need to create a rule for a group of addresses, for example, the entire subnet **54.246.198.0**: In fact, it is the range of addresses from **54.246.198.1** to **54.246.198.254**.&#x20;

Click '**Save settings**'.
