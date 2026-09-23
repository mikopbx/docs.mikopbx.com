---
description: Setting up non-working time rules
---

# Night and Holiday Switch

"Off-hours" in MikoPBX is a tool for setting up call processing rules during periods when the company is not working, such as at night, on weekends or holidays. With its help, administrators can determine how the system will handle incoming calls during off-hours: forward to an answering machine, play special voice messages or forward calls to the mobile numbers of on-duty employees. This ensures correct interaction with customers outside of working hours and maintains a high level of service.

<figure><img src="../../.gitbook/assets/1 (20).png" alt=""><figcaption><p>"Call routing" -> "Night and Holiday Switch" Section</p></figcaption></figure>

## Creating a rule

To add a new rule, click on the "**Add Time Interval**" button.

<figure><img src="../../.gitbook/assets/2 (11).png" alt=""><figcaption><p>"Add time interval" button</p></figcaption></figure>

A form for creating a new rule will open.

<figure><img src="../../.gitbook/assets/3 (20).png" alt=""><figcaption><p>A form for creating a new rule</p></figcaption></figure>

In the form, you will find the following fields:

* **Period**: The calendar period when employees are absent from the office, such as during New Year's or May holidays.
* **Weekdays**: Specific weekdays for which the rule will be applied.
* **Time Range**: The time period during the day when employees are absent.
* **Incoming Call Action**: You can choose to **play a sound file** or perform a call transfer. Call transfer options include transferring the call to a conference, IVR menu, queue, internal employee extension, or specific termination numbers.

In the **Note** field, you can add a note with a description of the created rule, so that you can quickly navigate through the essence of this rule using this description. With the **eraser button**, you can clear the fields opposite which this button is located.

### Apply only to certain incoming routes

By activating this function, a new menu "**Route restrictions**" will appear on top of you

<figure><img src="../../.gitbook/assets/7 (11).png" alt=""><figcaption><p>"Apply only to certain incoming routes" switch and "Route restrictions" Section</p></figcaption></figure>

Here you can choose which specific routes the rule you are creating will apply to.

<figure><img src="../../.gitbook/assets/8 (2).png" alt=""><figcaption><p>"Route restrictions" section</p></figcaption></figure>

## Examples of rules

This rule is used for calls during non-working hours from Monday to Friday, specifically from **7:00** **PM** to **9:00 AM:**

<figure><img src="../../.gitbook/assets/5 (1).png" alt=""><figcaption><p>Example of the rule</p></figcaption></figure>

This rule is used to handle calls on Saturdays and Sundays:

<figure><img src="../../.gitbook/assets/6 (6).png" alt=""><figcaption><p>Example of the rule</p></figcaption></figure>
