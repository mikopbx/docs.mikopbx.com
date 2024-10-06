# MikoPBX Manual

## Introduction

Thank You for choosing **MikoPBX ♥️**&#x20;

**MikoPBX** is a telephony server with an operating system and a convenient web interface. It works with almost any phone technology in the world. MikoPBX is based on **Asterisk** and has very low PC hardware requirements.

{% hint style="success" %}
Don't know where to start? Follow [these instructions](master/quick-start.md) to help you get MikoPBX up and running as quickly as possible.
{% endhint %}

## Structure of this manual

## Chapter 1. Installation

This chapter provides step-by-step instructions for installing MikoPBX on various platforms.

* [Separate Computer](setup/bare-metal.md)
* [Virtual Machine](setup/hypervisor/)
* [Cloud](setup/cloud/)
* [Docker container](setup/docker/)

## Chapter 2. Introduction to MikoPBX

First steps with MikoPBX. How to find out the IP address of your system, log in to the web interface and perform basic settings. [read more...](master/getting-to-know-mikopbx.md)

## Chapter 3. Configuring licensing in MikoPBX

After installing MikoPBX, it is necessary to configure the software license MIKO Sas License, without it it is impossible to work with additional modules. [read more...](manual/modules/pbx-extension-modules/licensing.md)

## Chapter 4. Telephony

In the **Telephony** section, you can set up a list of employees, set up an internal number for each employee, configure call routing and analyze call history.

* [Extensions](manual/telephony/extensions.md)
* [Call queue](manual/telephony/call-queues.md)
* [IVR Menu](manual/telephony/ivr-menu.md)
* [Conference rooms](manual/telephony/conference-rooms.md)
* [Sound Files](manual/telephony/sound-files.md)
* [Сall history](manual/telephony/call-detail-records.md)

## Chapter 5. Call  Routing

MikoPBX manages phone accounts, service providers. You will learn how to set up accounts in this chapter.

* [Telephony providers](manual/routing/providers.md)
* [Incoming routing](manual/routing/incoming-routing.md)
* [Outbound routing](manual/routing/outbound-routing.md)
* [Night and Holiday Switch](manual/routing/out-off-work-time.md)

## Chapter 6. Modules <a href="#glava_6_moduli" id="glava_6_moduli"></a>

* [Registration in the marketplace](manual/modules/pbx-extension-modules/licensing.md)
* [Extensions and plugins](manual/modules/pbx-extension-modules/)
* [Miko Modules](modules/miko/)
* [Application dialplans](manual/modules/dialplan-applications.md)

## Chapter 7. Maintenance <a href="#glava_7_obsluzhivanie" id="glava_7_obsluzhivanie"></a>

* [PBX Update ](manual/maintenance/update/)
* [System log entries](manual/maintenance/system-diagnostic.md)&#x20;
* [Reboot](manual/maintenance/restart.md)
* [Backup module](broken-reference)

## Chapter 8. Network and Firewall  <a href="#glava_8_set_i_firewall" id="glava_8_set_i_firewall"></a>

* [Network setting](manual/connectivity/network.md)
* [Network Firewall](manual/connectivity/firewall.md)
* [Anti brute force](manual/connectivity/fail2-ban.md)

## Chapter 9. System <a href="#glava_9_sistema" id="glava_9_sistema"></a>

* [General settings](manual/system/general-settings.md)
* [Date and Time ](manual/system/time-settings.md)
* [Mail settings](manual/system/mail-settings/)
* [Asterisk Manager Interface](manual/system/asterisk-managers.md)
* [System file customization ](manual/system/custom-files.md)

## FAQ <a href="#faq" id="faq"></a>

The section presents the solution of telephony problems that telephone exchange administrators most often face. By clicking on the links below, you can get acquainted with practical examples and find answers to frequently asked questions about MikoPBX.&#x20;

* [Setup](faq/setup/)
* [Managment](faq/management/)
* [Troubleshooting](faq/troubleshooting/)
* [Incoming Routing](faq/incoming-routing/)
* [Outbound routing](faq/outbound-routing/)
* [Integration with 1C ](faq/1c-integrations.md)
* [Scenarios and cases ](faq/cases/)
* [Configuring Providers ](faq/providers/)
* [Setting up softphones ](faq/softphones/)
* [VoIP gateways ](faq/voip-gateways/)
* [IP Phones](faq/ip-telefones/)
* [Interconnections](faq/interconnections/)
