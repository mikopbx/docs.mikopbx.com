# Call queue

Queues allow you to:

1. Distribute phone calls among a group of employees (agents): You can create a call queue and add multiple employees to it. When a call comes in, the system automatically routes it to an available employee in the queue, ensuring a more even distribution of workload and increasing call handling efficiency.
2. Hold the customer on the line when all employees are busy: If all employees in the queue are occupied with other calls, the customer will be placed on hold until one of the employees becomes available. This helps avoid call abandonment and ensures better customer service.
3. Notify the customer of their position in the queue and approximate wait time: While the customer is in the queue, the system can provide information about their current position in the queue and an estimated wait time. This helps keep the customer informed and improves their waiting experience.
4. Display the queue name along with the customer's number on the employee's phone: When an employee answers a call from the queue, their phone displays not only the customer's number but also the name of the corresponding queue. This helps the employee handle calls more effectively and provide personalized service.

To configure call queues in MikoPBX, go to the "**Telephony**" section and select "**Call Queue**." Here, you can create and customize your queues according to your business requirements and customer service needs.

<figure><img src="../../.gitbook/assets/1 (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
**The default call duration for a queue is set to 300 seconds (5 minutes)**. After this time limit is reached, the call will be automatically terminated. To bypass this limitation, you can configure "**Scenario 1**" as described in the instructions for "**Call Routing on Failures**".
{% endhint %}

## Main settings

To add a new queue, perform the "Add a new call queue" action

<figure><img src="../../.gitbook/assets/2 (19).png" alt=""><figcaption></figcaption></figure>

In the queue creation form or dialog, you will find the following fields:

* **Queue Name**: Enter a name for the queue. This name will be used for reference when setting up call routing rules.&#x20;
* **Note**: Provide a brief description or note about the queue. This information will be visible in the queue list, allowing you to provide additional details or instructions.

<figure><img src="../../.gitbook/assets/3.png" alt=""><figcaption></figcaption></figure>

## Queue Operators

In the **Queue Operators** section, you can add an arbitrary number of employees (queue agents) and specify a call distribution **strategy**.

<figure><img src="../../.gitbook/assets/4 (15).png" alt=""><figcaption></figcaption></figure>

Here are the options for queue strategy:

* _Ring All_: Calls are distributed to all **available agents** until someone answers the call (default behavior).
* &#x20;_Least Recent_: The call is routed to the **agent who has been idle for the longest time** within the queue.
* &#x20;_Fewest Calls_: The call is routed to the **agent who has handled the fewest number of calls** within the queue.&#x20;
* _Random_: A **random available agent** within the queue is selected to receive the call.&#x20;
* _Round Robin_: The call is distributed to each **agent in a sequential manner**, cycling through the list of agents.&#x20;
* _Memory Hunt_: The system remembers the **last agent who answered a call** and starts the distribution from that agent onwards.

&#x20;When setting up a queue, you can choose one of these strategies to determine how calls are distributed among the agents in the queue. The strategy you select will depend on your specific call handling requirements and the desired distribution behavior

## **Advanced Settings**

<figure><img src="../../.gitbook/assets/5 (9).png" alt=""><figcaption></figcaption></figure>

In this section, you can provide additional information:

* **Phone number for this queue** - you can call the queue using this number from any internal employee extension. Calls can also be transferred to this number.
* **Short name for the queue** - for display before the CallerID on the telephone device of the subscriber, for example, "consult."

<figure><img src="../../.gitbook/assets/6 (1).png" alt=""><figcaption></figcaption></figure>

### Queue settings for the agent

<figure><img src="../../.gitbook/assets/7 (6).png" alt=""><figcaption></figcaption></figure>

* **Time attempt call to agents**  - the duration in seconds for which a call will ring on an individual agent's phone. After this time elapses, the call to the agent will be logged as a missed call in the call history. Once the ring time is over, the call will be routed to the next available agent based on the selected strategy.
* **The rest time of the agent after the processing of the call, before starting to accept new calls** - the duration in seconds that is counted from the moment an agent finishes a call from the queue until they are ready to receive new calls. This period allows agents to update notes, complete necessary tasks, or take a short break before being assigned another call.
* **Receive New Calls During A Call** - this toggle switch enables or disables the ability to receive new calls while the agent is already on a call. When enabled, agents can handle multiple calls simultaneously.

### Queue settings for the caller

<figure><img src="../../.gitbook/assets/8 (12).png" alt=""><figcaption></figcaption></figure>

* **What the caller hears while waiting** - During the wait for their call to be answered, the caller can hear either hold music or a ringing tone.
* **Background Music** (MOH) - You can specify a unique audio file to be played to the caller during the wait, such as promotional materials.
* **Notify about current queue position** - If all operators (queue agents) are occupied, enabling this toggle switch allows you to notify the caller about their position in the queue. If the Additional Audio Announcement option is activated, this announcement will supplement the information about the position.
* **Notify about estimated hold time** - If all operators (queue agents) are occupied, enabling this toggle switch allows you to inform the caller about the approximate wait time for a call to be answered. If the Additional Audio Announcement option is activated, this announcement will supplement the information about the estimated wait time.
* **Additional notification** - A sound message is played only if all participants in the queue are occupied.
* **Time in seconds to repeat all alerts periodically** - Describes the interval at which to announce the queue position, wait time, and announcement.

### Call routing in case of failures

<figure><img src="../../.gitbook/assets/9 (8).png" alt=""><figcaption></figcaption></figure>

* **The script #1** - In this scenario, you can configure the maximum allowable wait time for a client in the queue. If none of the queue agents can answer the client within the specified time, you can set a number to which the call will be redirected.
* **The script #2** - If there are no agents available in the queue (meaning no agents are currently logged into the phone system), you can specify a number to which the client's call will be transferred.

{% hint style="info" %}
In these scenarios, as a redirection number, you can choose not only an internal extension but also options such as a conference, queue, IVR (Interactive Voice Response), or a special number within the dial plan application. These options provide flexibility in directing the call to different destinations based on your specific requirements or business needs.
{% endhint %}

{% hint style="warning" %}
**The default call duration for the queue is set to 300 seconds (5 minutes)**. If you require a longer interval, you can specify a higher duration in **Scenario 1** and provide a backup number to redirect the call to. This allows you to customize the wait time and ensure that if none of the queue agents can answer the call within the specified duration, it will be redirected to the designated backup number.
{% endhint %}

