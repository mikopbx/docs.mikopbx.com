# MikoPBX 2023.2

### Optimization for Small Displays

In the new version of MikoPBX, we've made several interface and form improvements, allowing administrators to manage the server from tablets or even mobile phones. We tested form functionality in various resolutions and made the sidebar menu dynamic. If the screen is small, the menu is hidden, and unnecessary fields in tables are also hidden.

<figure><img src="../../.gitbook/assets/2023_2AdaptiveEng.gif" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Adaptive forms</em></p></figcaption></figure>

### User Access Management

We made numerous changes to the MikoPBX code to enable user access management to different sections of the system, including hiding or showing form elements, menu items, and buttons. We also added support for multiple authentication methods and external module-based authentication, including domain login and password usage.

<figure><img src="../../.gitbook/assets/module-index-page.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Employee access control and multiple authentication</em></p></figcaption></figure>

You can read more about the user access management module in [its documentation](../../modules/miko/module-users-u-i.md).



### Active directory and LDAP synchronisation

The new domain synchronization module facilitates a bidirectional exchange of employee data and their contact numbers between the domain and MikoPBX. This module automates the data synchronization process, ensuring that the information in the domain remains up-to-date. When onboarding a new employee, their details are automatically integrated into the MikoPBX telephony system, with an available internal number being assigned.

<figure><img src="../../.gitbook/assets/ModuleLdapSync - index.png" alt=""><figcaption></figcaption></figure>

You can read more about the ldap sync module in [its documentation](../../modules/miko/module-ldap-sync.md).

### Additional Fields and Tabs from Modules

In the new version of MikoPBX, we expanded the API to manage the interface. Now, when you install additional modules, you can modify the web interface of existing forms, add tabs, buttons, and input fields.

For example, with the User Groups module, you can manage groups directly from the employee form.

<figure><img src="../../.gitbook/assets/New2023.2.additionalFieldsEn.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Additional fields from modules</em></p></figcaption></figure>

You can also manage access rights and authentication data for employees directly from their card.

<figure><img src="../../.gitbook/assets/New2023.2.additionalTabEn.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Additional tabs from modules</em></p></figcaption></figure>

### Code Comment Refactoring

In the MikoPBX source code, we've made significant improvements and changes to ensure that class, method, and function descriptions follow best practices for JS and PHP development. We've organized complex classes separately from simpler ones and restructured some algorithms to work in the background.

<figure><img src="../../.gitbook/assets/New2023.3.CodeCommentsEng.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Code comments and descriptions for each method and class</em></p></figcaption></figure>

### Enhanced Security

We've introduced background tasks in the system to regularly check the complexity of passwords for SIP, AMI, and system access. Additionally, we've updated the general settings form to prevent "peeking" at previously set passwords.

<figure><img src="../../.gitbook/assets/New2023.2 - simple passwords en.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Automatic detection of weak passwords for SIP, SSH, WEB, and AMI accounts</em></p></figcaption></figure>

### Call Recordings Retention Period

The system settings now allow you to set the retention period for call recordings. You can choose from several standard values or disable the deletion of old recordings. In this case, recordings will be deleted only if the storage space drops below 500 megabytes, and they will be deleted starting from the oldest ones.

<figure><img src="../../.gitbook/assets/New2023.2 - recordingsSliderEn.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Configuring the call recordings retention period</em></p></figcaption></figure>

### Docker Container Optimization

Installing MikoPBX inside a Docker container is one of the installation options. In the new release, we optimized the web interface and console menu, hiding menu items not used in the container installation.&#x20;

We've also improved network settings, allowing you to specify the system's external address, particularly useful for complex network topologies with port forwarding to public addresses on systems deployed within the perimeter and installed inside Docker containers.

<figure><img src="../../.gitbook/assets/New2023.2 - docker-nework-en.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Optimized network settings for the docker instalation</em></p></figcaption></figure>

### Customizing System Files with Scripts

In some cases, more complex modifications to system files are required than simply adding text to the end of a configuration file. For instance, you may need to redistribute PJSIP account parameters while retaining the ability to configure the system through the web interface.

We've introduced a new approach to customization, where you can describe a Bash script that will execute each time the system generates a configuration file. This way, integrators can make precise changes to configuration files without developing additional modules.

For example, you can modify the **pjsip.conf** file and change the _max\_contacts_ parameter for all internal numbers, except one.

<figure><img src="../../.gitbook/assets/New2023.2 - pjsip max contact en.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Modifying pjsip.conf with a script</em></p></figcaption></figure>

Or you can include additional lines of code within the dialplan in the **extensions.conf** file.

<figure><img src="../../.gitbook/assets/New2023.2 - extensions add string en.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Modifying extensions.conf with a script</em></p></figcaption></figure>

{% hint style="info" %}
This tool adds flexibility to the customization capabilities of the system but may lead to complete system malfunction. We strongly recommend testing customization scripts on a copy of the working system.
{% endhint %}

You can see the script's result on the file contents tab, after the system completes the generation and script execution. For some files, this process takes 1-2 minutes, while others may require system restart.

### Email Notifications for System Issues

The advice mechanism is now integrated with the notification mechanism. When changing the SSH password or encountering disk issues, the administrator will receive an email with details about the MikoPBX parameters in which the incident occurred. Previously, errors could only be checked after logging into the system, but now notifications are sent automatically.

In the future, additional metrics will be added to this system, including average CPU load, memory usage, issues with IP telephony provider registration, and critical kernel issues.

### Refactoring of the App Store

We've rewritten the marketplace's code, unified the tabs, and moved part of the modules to the backend. As you already know, MikoPBX is a free open-source system without any restrictions. We don't plan on changing this policy, but development requires resources. Therefore, we plan to monetize MikoPBX through the development and sale of our own and partner extensions in our app store.&#x20;

In the latest update, we've made some changes to the interface of the section, combining module management and system registration into separate tabs in one section. Paid and free modules are now marked with different icons in the list. We've also optimized the module installation code and fixed all identified errors.

<figure><img src="../../.gitbook/assets/New2023.2 - marketplace-en.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Updated marketplace UI</em></p></figcaption></figure>

Under the hood, MikoPBX hides a lot of changes and improvements that allow the development of functional extensions. If you're proficient in PHP and JS programming languages, understand how Asterisk works, and have ideas for developing new modules or are already actively doing so, we invite you to join the [developer channel on Telegram](https://t.me/mikopbx\_dev). Let's develop MikoPBX together!

### New Interface Languages: Romanian and Dutch

We're gradually expanding the set of basic translations for the web interface. In the new release, we've added 2 new languages and are improving the others.

<figure><img src="../../.gitbook/assets/New2023.2.RomanianLanguage.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Interface translation to Romanian</em></p></figcaption></figure>

<figure><img src="../../.gitbook/assets/New2023.2 - new translation Dutch.png" alt=""><figcaption><p><em>New in MikoPBX 2023.2: Interface translation to Dutch</em></p></figcaption></figure>

A huge thanks to our translators for their help:&#x20;

* Jochem Pluim&#x20;
* Secrieru Ion&#x20;
* Mikayil Isayev&#x20;
* Voutsas Theocharis&#x20;
* Everton Massen Goncalves&#x20;

If you want to help with the translation of the MikoPBX interface and modules, [follow this link](https://weblate.mikopbx.com/projects/mikopbx/admin-web-interface/).



## Clear Current Log Button

<figure><img src="../../.gitbook/assets/MikoPBX-erase-logs-en.png" alt=""><figcaption><p>New in MikoPBX 2023.2. Clearing the log from the interface</p></figcaption></figure>

When configuring calls, developing new applications, and analyzing issues, it's sometimes necessary to analyze system logs, which are available in the MikoPBX web interface. We have added a clear log file button that allows you to start your analysis with a clean slate.

### API for Programmatic Employee Creation

In the current release, an API for quickly creating a large number of employees has been implemented. During testing, we described the algorithm for generating new employees in ChatGPT and conducted stress testing for creating 700 random accounts in different languages. It took about 1 minute to complete the load test, and it was successful. You can read more about this case in detail [here](../../faq/cases/extensions-generation-by-rest-api.md).

<figure><img src="../../.gitbook/assets/Screenshot 2023-09-06 at 14.04.52.png" alt=""><figcaption><p>New in MikoPBX 2023.2. Programmatic generation of employee accounts</p></figcaption></figure>
