---
description: This tutorial will describe how to connect via SSH using Putty
---

# Connecting to PBX using SSH client (Putty)

1. Download the program to connect via SSH. This can be done on the official website at the [link](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
2. Run the downloaded program. The main menu will open for you.

<figure><img src="../../../.gitbook/assets/1 (21).png" alt=""><figcaption></figcaption></figure>

3. Go to the "**Connection**" - "**Data**" section

<figure><img src="../../../.gitbook/assets/2 (4).png" alt=""><figcaption></figcaption></figure>

4. "**Auto-login username**" specify **root**&#x20;

&#x20;      "**Terminal-tipe string**" specify **xterm-256color**

<figure><img src="../../../.gitbook/assets/3 (12).png" alt=""><figcaption></figcaption></figure>

5. Go to the "**Translation**" section

<figure><img src="../../../.gitbook/assets/4 (5).png" alt=""><figcaption></figcaption></figure>

6.  «**Remote character set**» - specify **UTF-8**

    Set the flag «**Enable VT100 line drawing even in UTF-8 mode**»

<figure><img src="../../../.gitbook/assets/5 (13).png" alt=""><figcaption></figcaption></figure>

7. Go to the section "**Session**" - "**Logging**". Here you can configure saving the output to a file:

<figure><img src="../../../.gitbook/assets/6 (2).png" alt=""><figcaption></figcaption></figure>

8. Go to the "**Session**" section

<figure><img src="../../../.gitbook/assets/7 (1).png" alt=""><figcaption></figcaption></figure>

9. Required data:

* **Host Name** (or IP address) — IP address of the PBX&#x20;
* **Port** — port for SSH connection by default 22
* Enter the session name and save its settings

<figure><img src="../../../.gitbook/assets/8 (1).png" alt=""><figcaption></figcaption></figure>

10. In the future, use the "**Load**" action to use the previously saved session

<figure><img src="../../../.gitbook/assets/9 (2).png" alt=""><figcaption></figcaption></figure>

11. Perform the "**Open**" action to connect to the PBX and enter the SSH password

<figure><img src="../../../.gitbook/assets/10 (4).png" alt=""><figcaption></figcaption></figure>

12. &#x20;Before connecting, you need to allow password authorization in the MikoPBX web interface, as well as set a password for connection: to do this, go to "**General Settings**" -> "**SSH**"

<figure><img src="../../../.gitbook/assets/14 (3).png" alt=""><figcaption></figcaption></figure>

13. &#x20;After entering the SSH password, the PBX menu will open

<figure><img src="../../../.gitbook/assets/11 (7).png" alt=""><figcaption></figcaption></figure>

14. &#x20;To open the console, go to "**\[9] Console(Shell)**"

<figure><img src="../../../.gitbook/assets/12.png" alt=""><figcaption></figcaption></figure>
