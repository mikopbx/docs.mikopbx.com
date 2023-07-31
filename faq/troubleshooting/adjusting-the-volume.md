# Adjusting the volume

There are cases when a customer can be heard very quietly during a phone call and there is no way to increase the volume level on the telephone /softphone. Let's describe a way that can help in solving the problem:

1. Go to "**System**" -> "**System file customization**"

<figure><img src="../../.gitbook/assets/1 (26).png" alt=""><figcaption></figcaption></figure>

2. Open **modules.conf** for editing

<figure><img src="../../.gitbook/assets/2 (28).png" alt=""><figcaption></figcaption></figure>

3. Add to the end of the file

```
load => func_volume.so
```

Save the changes

<figure><img src="../../.gitbook/assets/3 (16).png" alt=""><figcaption></figcaption></figure>

4. Open the **extensions.conf** file for editing

<figure><img src="../../.gitbook/assets/4 (2).png" alt=""><figcaption></figcaption></figure>

5. Add the following code to the end of the file:

```
[all_peers-custom]
exten => _X!,1,Set(__DYNAMIC_FEATURES=vUp#vDown)
    same => n,Set(VOLUME(TX)=5)
    same => n,Set(VOLUME(RX)=5)
    same => n,return   
    
[add-trim-prefix-clid-custom]
exten => _.X!,1,Set(__DYNAMIC_FEATURES=vUp#vDown)
    same => n,Set(VOLUME(TX)=5)
    same => n,Set(VOLUME(RX)=5)
    same => n,return

[volume-level-control]
exten => up,1,NoOp()
    same => n,ExecIf($[ "${Vol}x" == "x" ]?Set(Vol=0)
    same => n,Set(Vol=$[${Vol}+5])
    same => n,Set(VOLUME(TX)=${Vol})  
    same => n,return
exten => down,1,NoOp()
    same => n,ExecIf($[ "${Vol}x" == "x" ]?Set(Vol=0)
    same => n,Set(Vol=$[${Vol}-5])
    same => n,Set(VOLUME(TX)=${Vol})  
    same => n,return
```

Save the changes.

<figure><img src="../../.gitbook/assets/24.png" alt=""><figcaption></figcaption></figure>

6. Open the **features.conf** file for editing

<figure><img src="../../.gitbook/assets/6 (5).png" alt=""><figcaption></figcaption></figure>

7. Add the following code to the end of the file:

```
[applicationmap](+)
vUp => #1,self,Gosub,"volume-level-control,up,1"
vDown => #0,self,Gosub,"volume-level-control,down,1"
```

Save the changes

<figure><img src="../../.gitbook/assets/7 (9).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
The default volume will be higher, the value is 5 instead of 0. If desired, the employee can dial **#1 to increase** the volume or **#0 to decrease**
{% endhint %}
