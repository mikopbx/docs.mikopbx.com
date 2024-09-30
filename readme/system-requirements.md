# System requirements

### Network Channel Requirement

An example of calculating the required channel bandwidth for different codecs for 30 simultaneous calls. PBX supports the most popular codecs:

* **G.711** - 4.67 Mbps
* **GSM** - 1.68 Mbps
* **G.722** - 4.67 Mbps
* **G.729** - 1.38 Mbps&#x20;

{% hint style="info" %}
The calculation is approximate, when using the same codec on all devices connected to the PBX. Read more [here](https://www.asteriskguru.com/tools/bandwidth\_calculator.php).
{% endhint %}

### Minimum system requirements

{% hint style="warning" %}
We recommend using two hard drives for PBX deployment.
{% endhint %}

* **800 Mb** hard disk for the main system&#x20;
* A **50+ Gb** hard drive for recording conversations&#x20;
* 1 (2 cores) **x86-64** processor&#x20;
* **2 GB** of RAM&#x20;
* Network Adapter

{% hint style="success" %}
A PC with such parameters, in our tests, holds **38 simultaneous incoming calls** under the conditions:&#x20;

* 10 agents are connected to the queue (all online)&#x20;
* Every second a new call comes in&#x20;
* Music (MOH) is played to the client while waiting&#x20;
* Modules on the PBX is not installed
{% endhint %}

{% hint style="warning" %}
Approximately, **1 hour** of conversation takes up **14MB** of disk space.
{% endhint %}
