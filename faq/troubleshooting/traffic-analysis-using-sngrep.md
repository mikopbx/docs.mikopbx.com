# Traffic Analysis Using Sngrep

**Sngrep** is a command-line tool for capturing and analyzing SIP traffic. It allows you to visualize SIP sessions, filter them, and track issues in voice connections.

{% hint style="info" %}
Use this application to analyze logs and send them to technical support.
{% endhint %}

To start working with the application, follow the SSH connection to the PBX [guide](connecting-to-a-pbx-using-an-ssh-client.md).

To **start** the application, use the command:

```bash
sngrep -r
```

{% hint style="success" %}
If multiple network interfaces are used, specify the interface ID when launching the application:

```bash
bashCopy codesngrep -d eth1 -r
```

The **-r** key allows capturing audio traffic.
{% endhint %}

You can view the list of interfaces using the following command:

<pre class="language-php"><code class="lang-php"><strong># ifconfig
</strong>eth0      Link encap:Ethernet  HWaddr 00:0C:29:08:EF:FD  
          inet addr:172.16.156.223  Bcast:172.16.156.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:81838 errors:0 dropped:0 overruns:0 frame:0
          TX packets:38019 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:66203565 (63.1 Mb)  TX bytes:7603334 (7.2 Mb)

eth1      Link encap:Ethernet  HWaddr 00:0C:29:08:EF:07  
          inet addr:172.16.32.162  Bcast:172.16.32.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:48506 errors:0 dropped:4432 overruns:0 frame:0
          TX packets:5386 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3698996 (3.5 Mb)  TX bytes:1886690 (1.7 Mb)

</code></pre>

Example of Sngrep Interface:

<figure><img src="../../.gitbook/assets/exampleOfView.png" alt=""><figcaption><p>The view of Sngrep</p></figcaption></figure>

The application window displays a list of all SIP dialogues:

* Use the **⇑** and **⇓** arrows to navigate between dialogues.
* Press **Enter** to view detailed information about a dialogue.

<figure><img src="../../.gitbook/assets/statusOfSIP.png" alt=""><figcaption><p>information about the dialogue</p></figcaption></figure>

* In the detailed view, you can examine specific SIP packets by selecting them with **⇑** and **⇓**.
* Press **Enter** to view the contents of a SIP packet.

<figure><img src="../../.gitbook/assets/statusOfSIP2.png" alt=""><figcaption><p>Contents of the SIP packet</p></figcaption></figure>

* Press **ESC** to return to the previous window.
* Use the **Space** key to select multiple SIP dialogues and press **Enter** to view them in one window.
* In the detailed view, use the **Space** key to select two **SIP** packets for comparison.

<figure><img src="../../.gitbook/assets/comparasion.png" alt=""><figcaption><p>Comparison of two SIP packages</p></figcaption></figure>

## Saving a Dump <a href="#soxranenie_dampa" id="soxranenie_dampa"></a>

1. Use the **Space** key to select the SIP dialogue "Call" of interest.

<figure><img src="../../.gitbook/assets/callDialogue.png" alt=""><figcaption><p>Dialogue "Call"</p></figcaption></figure>

2. Press **F2** to open the save dump dialogue:

* Use the **⇑** and **⇓** arrows to navigate between form fields.
* Enter the path and file name.
* Select the save action and press **ENTER**.
* Download the file using [SSH connection to the PBX with WinSCP](connecting-to-a-pbx-using-winscp.md).

#### Filtering <a href="#filtracija" id="filtracija"></a>

1. Press **F7** to open the filter dialogue:

<figure><img src="../../.gitbook/assets/filterOption.png" alt=""><figcaption></figcaption></figure>

2. Use the **⇑** and **⇓** arrows to navigate between form fields.
3. Use the **Space** key to select SIP methods for analysis.
4. Select the **Filter** action and press **ENTER**.
