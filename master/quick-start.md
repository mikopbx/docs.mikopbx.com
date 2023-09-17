# Quick start

This guide will help you get MikoPBX up and running as quickly as possible. Follow the instructions step-by-step in the order in which they are presented.

{% hint style="success" %}
The contents of the complete MikoPBX documentation are available at the [link](../).
{% endhint %}

### Step 1. Installing MikoPBX

{% hint style="danger" %}
Attention! Before installing MikoPBX, be sure to read the [system requirements](system-requirements.md)!
{% endhint %}

**MikoPBX** is a full-fledged operating system for your hardware, it is not a separate program. It is delivered as an image (\*.iso, \*.img, \*.raw file).&#x20;

There are several options for installing MikoPBX. Choose the most suitable installation option for yourself:

* Installing a PBX on a **virtual machine.** [Installation instructions. ](../setup/hypervisor/)
* Installing a PBX in a public **cloud**. [Installation instructions. ](../setup/cloud/)
* Installing MikoPBX from **USB** to a separate dedicated server. The PC must support booting from USB. [Installation instructions. ](../setup/bare-metal.md)
* Installing MikoPBX in a **Docker** container. [Installation instructions.](../setup/docker.md)

### Step 2. The first launch of the PBX!

After successful installation of the PBX, you need to go to its settings in the web interface. Follow the [instructions ](getting-to-know-mikopbx.md)to perform the first launch of the PBX in the web interface.

### Step 3. Configuring the network interface <a href="#shag_4_nastrojka_setevogo_interfejsa" id="shag_4_nastrojka_setevogo_interfejsa"></a>

For stable operation of the PBX, you need to configure the network through the **Network and Firewall → Network Interfaces section**. Follow the [instructions ](../manual/connectivity/network.md)to set up.

### Step 4. Configuring the Firewall

In MikoPBX, all local subnets can be described in the **Network and Firewall → Firewall** section. The firewall is designed to restrict access to the station by traffic type and subnets. Perform the setup according to the [instructions](../manual/connectivity/firewall.md).

### Step 5. Setting up Fail2ban

Fail2ban blocks IP addresses with non-standard activity, it is able to reduce the speed of unsuccessful authentication attempts, and allows you to protect your PBX from hacking. Perform the setup according to the [instructions](../manual/connectivity/fail2-ban.md).

### Step 6. Setting Up Employee Accounts

At this stage, we are creating accounts for internal employee numbers. Follow these [instructions ](../manual/telephony/extensions.md)to create a list of internal numbers.

### Step 7. Connecting providers

To connect the provider to MikoPBX, follow these [instructions](../manual/routing/providers.md).

### Step 8. Configuring Routing

At this stage, it is necessary to set the routing rules for incoming and outgoing calls, how calls coming through a certain provider will be handled.

* [Incoming routing](../manual/routing/incoming-routing.md)
* [Outbound routing](../manual/routing/outbound-routing.md)

To create routing rules, you may need:

* [Call queue](../manual/telephony/call-queues.md)
* [IVR Menu](../manual/telephony/ivr-menu.md)
* [Conference rooms](../manual/telephony/conference-rooms.md)

This completes the basic setup of MikoPBX! For a deeper study of the capabilities of MikoPBX, we recommend that you refer to the [documentation](../).

### Step 9. Register in the Marketplace

To use the various modules, you need to register in the Marketplace. You can read about how to do this [here](../manual/modules/pbx-extension-modules/licensing.md).
