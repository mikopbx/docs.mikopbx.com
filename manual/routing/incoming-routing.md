# Incoming routing

## General information

<figure><img src="../../.gitbook/assets/IncomingRoutingSection.png" alt=""><figcaption><p>Incoming Routing section</p></figcaption></figure>

In this section, you need to create rules and templates for distributing incoming calls for providers created in MikoPBX. The rules for incoming calls describe the route of a call from the moment it arrives at the PBX to the moment it is completed. You can create an unlimited number of inbound routing rules. You can create several rules for one provider.

{% hint style="info" %}
Additional examples of configuring incoming routing are available in the [FAQ ](broken-reference)section.
{% endhint %}

## Routing rule priority and default route

Rules are listed in order of priority. If no one answers the incoming call within the time interval specified in the rule, the call will be routed to the next priority rule. Rules can be moved up and down in the list, that is, their priority can be changed by dragging them by the arrows.

<figure><img src="../../.gitbook/assets/PriorityScheme.png" alt=""><figcaption><p>Priority Scheme </p></figcaption></figure>

If the call is not answered according to any of the rules, the **Default incoming route** is used.

<figure><img src="../../.gitbook/assets/defaultRoute.png" alt=""><figcaption><p>Default incoming route</p></figcaption></figure>

The following actions are available and can be specified as the default rule:

* **Play busy signa**l - the client will play a busy signal and the incoming call will be ended;&#x20;
* **Hang up**;&#x20;
* **Redirect the call** - the call can be transferred to a number that you can select in the field located to the right of the action. You can select an IVR menu, call queue, conference, or employee extension number as the number for transfer.

## Multiple routes for one provider

For one provider, you can describe **several** incoming routes.&#x20;

First, the call goes along the upper route. If the client does not get through, then the call goes according to the lower rule (lower priority). If the client does not get through via the second route, then the call goes through the **default route.**

<figure><img src="../../.gitbook/assets/Priority (1).png" alt=""><figcaption><p>Several incoming routes for one provider</p></figcaption></figure>

## Create a routing rule

To add a new incoming routing rule, click the **Add a new rule** button.

<figure><img src="../../.gitbook/assets/newRule (1).png" alt=""><figcaption><p>New Rule</p></figcaption></figure>

In the **Note** field, describe the route you want to implement. In the future, this will help you debug the call circuit.&#x20;

Select the **Provider** for which you are creating a new incoming call distribution template.&#x20;

The additional **DID number** is the number the client called you on. This field is optional and should be completed if you need to route calls more accurately.&#x20;

<figure><img src="../../.gitbook/assets/parameters1.png" alt=""><figcaption><p>Parameters for a new rule</p></figcaption></figure>

At the next step, you need to indicate to which **phone number** the incoming call from the client will be sent. The telephone number can be IVR menu numbers, call queues, conferences, or employee internal numbers.

<figure><img src="../../.gitbook/assets/parameters2.png" alt=""><figcaption><p>Parameters for a new rule</p></figcaption></figure>

Specify the time during which the call will be sent to the phone number you specified.

<figure><img src="../../.gitbook/assets/parameters3.png" alt=""><figcaption><p>Parameters for a new rule</p></figcaption></figure>

If after the specified time interval no one answers the incoming call, the call will be routed to the next priority rule.
