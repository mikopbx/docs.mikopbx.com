# Reset to factory settings

## Method 1

1. Go to "**General Settings**" -> "**Delete all system settings**"

<figure><img src="../../.gitbook/assets/4 (23).png" alt=""><figcaption></figcaption></figure>

2. In the input field, paste the text "**delete everything**", click "**Save settings**"

<figure><img src="../../.gitbook/assets/5.png" alt=""><figcaption></figcaption></figure>

## Method 2

1. Open the MikoPBX console menu. Use the keyboard to enter **9** to go to the **PBX console.**

<figure><img src="../../.gitbook/assets/3 (24).png" alt=""><figcaption></figcaption></figure>

2. Enter two commands sequentially:

```
cp /conf.default/mikopbx.db /cf/conf/mikopbx.db
reboot
```

3. After executing these commands, MikoPBX will reboot. The login to the web interface takes place with the login (admin) and password (admin) by default.
