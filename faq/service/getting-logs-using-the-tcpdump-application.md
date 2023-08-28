# Getting logs using the tcpdump application

1. Connect to your PBX via SSH ([instructions](../troubleshooting/connecting-to-a-pbx-using-an-ssh-client.md))
2. Run the command:

```
tcpdump -i eth0 -n -s 0 -vvv -w /tmp/capturefilename.pcap
```

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption><p>Command in the SSH interface</p></figcaption></figure>

3. Reproduce your situation, make a phone call. Next, press **CTRL + C** in the SSH console. The tcpdump application will be completed.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption><p>The result of tcpdump</p></figcaption></figure>

4. Connect to the PBX using WinSCP ([instructions](../troubleshooting/connecting-to-a-pbx-using-winscp.md))
5. Send the call log **/tmp/capturefilename.pcap** to technical support
