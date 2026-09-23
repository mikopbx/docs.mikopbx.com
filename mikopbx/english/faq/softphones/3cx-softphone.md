---
description: Instructions for configuring and connecting the 3CX Softphone to MikoPBX
---

# 3CX Softphone

**3CX** is a SIP softphone designed for business, offering voice and video calls as well as multichannel call and conference capabilities.

<figure><img src="../../.gitbook/assets/3cxSoftPhone.jpg" alt=""><figcaption><p>3CX softphone interface</p></figcaption></figure>

1. **Download and install** the 3CX Softphone from the official website ([link](https://www.3cx.com/voip/softphone/)).

<figure><img src="../../.gitbook/assets/mainpage.jpg" alt=""><figcaption><p>Start page</p></figcaption></figure>

2. **Add a new SIP account** by clicking "**New**":

<figure><img src="../../.gitbook/assets/newConnection.jpg" alt=""><figcaption><p>New SIP-connection</p></figcaption></figure>

3. **Enter the required data** for your SIP connection:

* **Account name** – any arbitrary name
* **CallerID** – employee’s name
* **Extension** – employee’s internal extension number (from the employee card in MikoPBX)
* **ID** – same as the internal extension number (from the employee card)
* **Password** – SIP password (from the employee card)

<figure><img src="../../.gitbook/assets/credentials.jpg" alt=""><figcaption><p>Connection credentials</p></figcaption></figure>

4. In the **"My location"** section, specify your MikoPBX IP address.

* If the softphone is on the **same local network** as MikoPBX, select **"I am in the office - local IP."**
* If you want to connect via the external IP address, choose **"I am out of the office - external IP."**

After entering these details, click **"OK."**

<figure><img src="../../.gitbook/assets/myLocation.jpg" alt=""><figcaption><p>"My location" section</p></figcaption></figure>

5. **Select the added account** and click **"OK"**:

<figure><img src="../../.gitbook/assets/connection.jpg" alt=""><figcaption><p>Steps for choosing SIP connection</p></figcaption></figure>

The softphone will then connect. You can verify the connection by checking the employee’s status indicator in MikoPBX:

<figure><img src="../../.gitbook/assets/successfulConnection (1).jpg" alt=""><figcaption><p>Successful connection!</p></figcaption></figure>
