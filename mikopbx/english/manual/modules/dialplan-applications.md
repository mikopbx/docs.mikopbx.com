---
description: Creating and Configuring Dialplan Applications
---

# Application dialplans

Dialplan applications are programmable voice applications in PHP and Asterisk Dialplan. MikoPBX comes with several pre-configured applications. With some basic knowledge of Asterisk dialplans, additional applications can be easily created. Like a phone account, applications can have an extension assigned in the settings.

MikoPBX comes with several pre-configured applications. With some basic knowledge of Asterisk dialplan, you can easily create additional applications. Like a phone extension, applications can have an internal number assigned in the settings.

Below you will see a description of the basic applications included in MikoPBX:

<figure><img src="../../.gitbook/assets/1 (2).png" alt=""><figcaption><p>List of basic Application dialplans</p></figcaption></figure>

| Application Number | Application Description                                                                                                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 000063             | The application reads the internal number of the employee used to call the application and voices it to the employee, i.e. the employee is voiced his internal number on the PBX                        |
| 000064             | **0000MILLI** - Generates a constant sound signal with a frequency of 1000 Hz. Used to check the quality of the connection.                                                                             |
| 10003246           | The **Echo** application sends the received audio signals back to the user so that the delay duration can be determined. In general, you hear what you say. The application is mainly used for testing. |

## Creating applications

MikoPBX applications are created from several plans of the Asterisk application suite. There are many examples of ready-to-run applications in the system. To add a new MikoPBX application, click on "**Add a New**" in the application menu.

<figure><img src="../../.gitbook/assets/2 (12).png" alt=""><figcaption><p>"Add a new" button</p></figcaption></figure>

In just a few steps, you can create your own applications. First, define the **Name and Call Number** for the application, and optionally fill in the **Comment field**.

Possible application code types:

* **PHP-AGI script** - AGI is an embedded method in Asterisk for executing external scripts (similar to CGI for HTTP servers), which can extend Asterisk's functionality using other programming languages, particularly PHP. AGI scripts can control call handling in the dialplan and are invoked from the extensions.conf file.
* **Asterisk Dialplan** - The configuration of the dialplan is contained in the Asterisk configuration file called **extensions.conf**. This is one of the most important configuration files where the processing and routing of incoming and outgoing calls are defined. This file governs the behavior of all connections passing through your PBX (Private Branch Exchange).

<figure><img src="../../.gitbook/assets/3 (6).png" alt=""><figcaption><p>Parameters of the new dialplan application</p></figcaption></figure>

Let's clarify: we will refer to MikoPBX applications as "applications" and Asterisk dialplan functions as "functions". For example, Answer(), NoOP(), Set(), and Wait() are functions. These are individual target functions in Asterisk that are then combined in MikoPBX to create more powerful MikoPBX applications.

Describe the logical operations in the text field of the **Programme Code**. Please note that only one command is allowed per line, for example:

<figure><img src="../../.gitbook/assets/4 (4).png" alt=""><figcaption><p>"Programme code" section</p></figcaption></figure>

The figure shows an example of the simplest application for the number 000063. After dialing the number, you will hear the robot voice your internal number.

{% hint style="danger" %}
MikoPBX will check the commands used. It is possible that incorrectly programmed operations may affect the performance of your telephone system.
{% endhint %}

Description of Asterisk functions that you can use in your applications:

| Наименование команды | Описание                                                                                                                                                                                             |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| answer               | Transfer the call to the answered state.                                                                                                                                                             |
| channel status       | Returns the status of the connected channel.                                                                                                                                                         |
| control stream file  | Sending a preset audio file to the channel, with the ability to control its playback (pause/rewind/resume playback) using the DTMF digits received from the subscriber, if specified. (Asterisk 1.2) |
| database del         | Deleting a key/value from the database.                                                                                                                                                              |
| database deltree     | Deleting the key/value tree from the database.                                                                                                                                                       |
| database get         | Get the value from the database.                                                                                                                                                                     |
| database put         | Adding/changing a value in the database.                                                                                                                                                             |
| exec                 | Execution of the specified Command. (Commands are functions that you use when describing the set plan in the extensions.conf file).                                                                  |
| get data             | Get data from the channel.                                                                                                                                                                           |
| get option           | Behaves similarly to the "STREAM FILE" command, but is used with a specified value for timeout. (Asterisk 1.2)                                                                                       |
| get variable         | Get the value of the channel variable.                                                                                                                                                               |
| hangup               | Break the connection (Hangup) on the current channel.                                                                                                                                                |
| noop                 | An empty command. Does nothing.                                                                                                                                                                      |
| receive char         | Accepts one character from the channel if it supports this feature.                                                                                                                                  |
| receive text         | Accepts a text string from a channel if it supports this feature.                                                                                                                                    |
| record file          | Writes to the specified file.                                                                                                                                                                        |
| say alpha            | Pronounces the specified string of characters. (Asterisk 1.2)                                                                                                                                        |
| say date             | Pronounces the date. (Asterisk 1.2)                                                                                                                                                                  |
| say datetime         | Pronounces the date and time according to the specified format. (Asterisk 1.2)                                                                                                                       |
| say digits           | Pronounces the specified string of digits.                                                                                                                                                           |
| say number           | Pronounces the specified number.                                                                                                                                                                     |
| say phonetic         | Pronounces the specified string of characters.                                                                                                                                                       |
| say time             | Pronounces the time.                                                                                                                                                                                 |
| send image           | Sends the image to the channel if it supports this feature.                                                                                                                                          |
| send text            | Sends text to the channel if it supports this feature.                                                                                                                                               |
| set autohangup       | Automatic termination of the connection (Autohangup) on the channel at the specified time.                                                                                                           |
| set callerid         | Setting the caller id for the current channel.                                                                                                                                                       |
| set context          | Setting the context for the current channel.                                                                                                                                                         |
| set extension        | Change the extension for the current channel.                                                                                                                                                        |
| set music            | Включение/Выключение музыки ожидания (Music on hold), например: «SET MUSIC ON default».                                                                                                              |
| set priority         | Enabling/Turning off the standby music (Music on hold), for example: "SET MUSIC ON default".                                                                                                         |
| set variable         | Setting the channel variable.                                                                                                                                                                        |
| stream file          | Sending an audio file to the channel.                                                                                                                                                                |
| tdd mode             | Setting the TDD mode for a channel that can support it to enable interaction with TDD.                                                                                                               |
| verbose              | Writing a message to the verbose log of the asterisk server.                                                                                                                                         |
| wait for digit       | Waiting for the DTMF button to be pressed                                                                                                                                                            |
