# An example of the implementation of a typical route of incoming calls

## Formulation of the problem

The client calls the company, a welcome message (voice greeting) sounds.&#x20;

During the playback of the voice greeting, the client can dial any internal number, for example, the extension number of an employee or the extension number of a department.&#x20;

After playing the voice greeting, the client can enter numbers from the phone:

* **1** - in this case, the call will be sent to the sales department. The **sales department** is a call queue consisting of two queue agents (two employees). When calling the sales department, the call must be received on the phones of employees at the same time. If none of the employees answered the customer's call within 25 seconds, the call should be sent to the employees' mobile numbers.&#x20;
* **2** - the call will be sent to the technical support department. The **Technical Support** Department is a call queue consisting of three queue agents (three employees). The call must be received by any available employee (queue agent). If the client waits more than 30 seconds for his call to be answered, the call must be transferred to the secretary. If the client has not entered anything / incorrectly dialed the number, then a repeated voice notification occurs and the client can enter the number again. As soon as all attempts to enter the correct number for the client are completed, the call is forwarded to the secretary.

## Solution

### Call queues

{% hint style="info" %}
Instructions on call queues are available at the [link](../../manual/telephony/call-queues.md).
{% endhint %}

For our example, you need to create three call queues:&#x20;

1. Sales Department (mobile numbers of employees)&#x20;
2. Sales Department (internal employee numbers)&#x20;
3. Technical Support Department

{% hint style="warning" %}
You must first create internal accounts for employees according to the [instructions ](../../manual/telephony/extensions.md)indicating their mobile numbers.
{% endhint %}

1. Go to the **Telephony** → **Call queue**. Click on the "**Add a new call queue**" button.

<figure><img src="../../.gitbook/assets/newCallQueue.png" alt=""><figcaption><p>Creating a new queue</p></figcaption></figure>

2. Creating a call queue for the sales department (for employees' **mobile** numbers).

<figure><img src="../../.gitbook/assets/firstQueue.png" alt=""><figcaption><p>Queue for the sales department(mobile)</p></figcaption></figure>

3. Creating a call queue for the sales department (for **internal** employee numbers).

<figure><img src="../../.gitbook/assets/secondQueue.png" alt=""><figcaption><p>Queue for the sales department</p></figcaption></figure>

4. We indicate that if none of the employees answered the customer's call within 25 seconds, the call should be sent to the employees' mobile numbers.

<figure><img src="../../.gitbook/assets/secondQueueExtra.png" alt=""><figcaption><p>Scenario in case of failure</p></figcaption></figure>

5. Creating a call queue for the technical support department.

<figure><img src="../../.gitbook/assets/thirdQueue.png" alt=""><figcaption><p>Queue for Technical Support department</p></figcaption></figure>

### IVR Menu

{% hint style="info" %}
Instructions for the IVR menu are available at the [link](../../manual/telephony/ivr-menu.md).
{% endhint %}

1. Go to **Telephony** → **IVR menu**. Click on the "**Add new IVR menu**" button.

<figure><img src="../../.gitbook/assets/NewIVR.png" alt=""><figcaption><p>Creating a new IVR menu</p></figcaption></figure>

2. Specify the IVR settings according to our description.

<figure><img src="../../.gitbook/assets/IVRMenu (3).png" alt=""><figcaption><p>IVR Menu</p></figcaption></figure>

### Incoming route

{% hint style="info" %}
Instructions for incoming routes are available at the [link](../../manual/routing/incoming-routing.md).
{% endhint %}

1. At the last step, we create an incoming call route for our provider. In the **Call Routing** → **Incoming routing** add a new rule.

<figure><img src="../../.gitbook/assets/NewRule (1).png" alt=""><figcaption><p>Section "Incoming routes"</p></figcaption></figure>

2. Fill in the data according to the template:

<figure><img src="../../.gitbook/assets/parametersOfRule (1).png" alt=""><figcaption><p>New incoming call handling rule</p></figcaption></figure>
