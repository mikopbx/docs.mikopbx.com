---
description: Description and configuration of network interfaces
---

# Network interface

The "**Network Interface**" section in MikoPBX is an interface for configuring the system's network connection parameters. Here, administrators can manage IP addresses, subnet masks, gateways, and other network settings for each network interface. This allows MikoPBX to be correctly integrated into the organization's network and ensure its stable operation in accordance with the requirements of the network infrastructure.

The section is located in "**Network and Firewall**" -> "**Network Interface**":

<figure><img src="../../.gitbook/assets/networkInterface.png" alt=""><figcaption><p>"Network Interface" Section in MikoPBX system</p></figcaption></figure>

## General parameters

The hostname is the name of the machine. If no value is specified, the default hostname used is 'mikopbx.local'.

<figure><img src="../../.gitbook/assets/1 (14).png" alt=""><figcaption></figcaption></figure>

## Network interfaces

There are two ways to configure the IP address:

1. **DHCP (Dynamic Host Configuration Protocol)** can be used for automatic IP address configuration. Enable the 'Use DHCP to obtain network settings' switch. This is recommended for most users. To not rely on DHCP server settings (to provide a specific address), you can disable the switch.
2. If you do not want to use settings obtained from a DHCP server, you can **configure the network manually**. This requires some knowledge about the network topology. To the right of the IP address, there is a field for Subnet Mask in CIDR format. You should use the alternative format: /8 corresponds to the subnet mask 255.0.0.0, /16 corresponds to 255.255.0.0, and /24 corresponds to 255.255.255.0.

"**VLAN ID**" - MikoPBX supports virtual network interfaces. This is relevant only for physical PCs. Sometimes a PC may have only one network interface, and it may not be possible to connect a second one physically. Using VLAN, you can create a virtual interface that works 'on top' of the physical one. One of the advantages of using VLAN is that all phone calls can be routed through it, while the network equipment can 'tag' all VLAN traffic and guarantee a stable connection.

The number of network interfaces in MikoPBX is not limited.

<figure><img src="../../.gitbook/assets/2 (13).png" alt=""><figcaption><p>"Network interfaces" section</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/3 (26).png" alt=""><figcaption><p>"Network topology" section</p></figcaption></figure>

{% hint style="info" %}
When enabling the option 'This station is located behind a NAT router,' it is mandatory to specify the external address or hostname of the router. Additionally, you need to perform port forwarding on the router for SIP port 5060 and RTP ports 10000-10200 to the local address of the PBX.

If your provider allows registration and you do not need to connect external subscribers, you can choose not to enable the option "**This PBX is located behind a NAT router",** even if the PBX is behind a NAT router.
{% endhint %}

## Manual configuration of network routes

Static routes are used when MikoPBX must send traffic to a specific network through a separate gateway instead of the main Internet gateway. This is most often required in the following cases:

* the PBX has several network interfaces;
* telephony, VPN, or remote offices are available through a separate router;
* traffic must be routed to a specific provider or branch-office subnet;
* the main gateway is used for Internet access, while separate internal networks must be reached through another gateway.

### Where to find this setting in the interface

1. Go to **Network and Firewall** → **Network interfaces**.
2. At the bottom of the form, find the **Static routes** block.

<figure><img src="../../.gitbook/assets/network-static-routes-section.png" alt=""><figcaption><p>Static routes configuration section</p></figcaption></figure>

{% hint style="info" %}
In Docker installations, the **Static routes** block may be hidden because the network stack is managed by the container or the host system.
{% endhint %}

### How to add a route

1. In the **Static routes** block, click **Add route**.
2. Fill in the route row.
3. Add several routes if needed. You can change the order by dragging rows; this order is saved as the route priority.
4. Click **Save** at the bottom of the form.

<figure><img src="../../.gitbook/assets/network-static-routes-add-route.png" alt=""><figcaption><p>Adding a static route</p></figcaption></figure>

### Field description

| Field | Description |
| ----- | ----------- |
| **Network** | Destination network address. For example, specify **192.168.10.0** for a remote subnet. If the route is needed for one specific IP address, specify that IP address and select the **32 - 255.255.255.255** mask. |
| **Mask** | CIDR mask of the destination network. It is selected from a list in the interface, for example **24 - 255.255.255.0** for a /24 subnet or **32 - 255.255.255.255** for a single address. |
| **Gateway** | IP address of the router through which MikoPBX should send traffic to the specified network. The gateway must be reachable from the selected interface. |
| **Interface** | Network interface through which the route will be used. You can leave **Auto** so that the operating system chooses the interface, or select a specific interface such as **eth0** or a VLAN interface. |
| **Description** | Administrator comment. Use it to describe the purpose of the route, for example **Branch VPN** or **SIP provider network**. |

Example route to a remote subnet:

| Field | Value |
| ----- | ----- |
| **Network** | **192.168.10.0** |
| **Mask** | **24 - 255.255.255.0** |
| **Gateway** | **172.16.32.15** |
| **Interface** | **eth0** or **Auto** |
| **Description** | **Office VPN** |

Example route to a single address:

| Field | Value |
| ----- | ----- |
| **Network** | **54.246.198.136** |
| **Mask** | **32 - 255.255.255.255** |
| **Gateway** | **172.16.32.15** |
| **Interface** | **eth0** or **Auto** |

### How to check that the route works

Check that the required remote address or service is reachable from MikoPBX: SIP provider registration, connection to a remote phone, access to a VPN network, or another working scenario for which the route was added.

If you have console or SSH access, you can additionally check which path the system selects:

```bash
ip route get 192.168.10.10
```

The output should show the expected gateway or interface. To check host reachability, you can use:

```bash
ping 192.168.10.10
```

You can also open **Maintenance** → **System logs** → **System information** and check the current network settings.

### Common mistakes

* **Invalid gateway.** The **Gateway** field must contain a valid IP address. If it is incorrect, MikoPBX may show the **Invalid gateway address** message.
* **The gateway is not in the selected interface subnet.** If a specific **Interface** is selected, make sure the specified gateway is reachable through that interface. Otherwise, the route may be saved, but traffic will not go through the required path.
* **Conflict with the main gateway.** Do not use a static route as a replacement for the main Internet gateway. The main gateway is configured in the Internet interface settings, while static routes are better suited for separate networks and addresses.
* **Invalid mask/CIDR.** Specify the network address and the correct **Mask**. For example, for the **192.168.10.0/24** subnet, use **192.168.10.0** and **24 - 255.255.255.0**; for a single IP address, use **32 - 255.255.255.255**.
* **The route is not applied after saving.** Check that the page was saved without errors, the route remains in the **Static routes** table, the selected gateway is reachable from MikoPBX, and **Interface** is selected correctly or set to **Auto**.
